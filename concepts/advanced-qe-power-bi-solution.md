# Advanced Quality Engineering Power BI Solution

This guide provides a sophisticated approach to building a Quality Engineering (QE) dashboard using Azure DevOps (ADO) OData feeds. It focuses on actionable insights, advanced data modelling, and specific metrics for defect leakage, automation coverage, and test velocity.

## 1. Core Objectives

*   **Data Extraction**: Pull data from Work Items, Test Results, and Pipelines via OData.
*   **Transformation Logic**: Calculate Defect Leakage, Automation Coverage, and Test Execution Velocity.
*   **Advanced Modelling**: Create a Star Schema connecting Test Runs to Requirements for full traceability.
*   **Key Metrics**: Requirement Traceability, Defect Density, Flakiness Index, MTTR.

## 2. Data Extraction (OData Feeds)

Connect to the following OData feeds using Power BI's OData connector. Replace `{Org}` and `{Project}` with your actual values.

**Base URL**: `https://analytics.dev.azure.com/{Org}/{Project}/_odata/v3.0-preview/`

### 2.1. Work Items (Requirements & Bugs)

```powerquery
Source = OData.Feed("https://analytics.dev.azure.com/{Org}/{Project}/_odata/v3.0-preview/WorkItems?"
    & "$select=WorkItemId, Title, WorkItemType, State, Reason, Severity, Priority, StoryPoints, ClosedDate, CreatedDate, Area/AreaPath, Iteration/IterationPath"
    & "&$expand=Area,Iteration"
    & "&$filter=WorkItemType eq 'User Story' or WorkItemType eq 'Bug'"
)
```

### 2.2. Test Runs & Results

```powerquery
Source = OData.Feed("https://analytics.dev.azure.com/{Org}/{Project}/_odata/v3.0-preview/TestRuns?"
    & "$select=TestRunId, Title, StartedDate, CompletedDate, TotalTests, PassedTests, FailedTests, IncompleteTests, Build/BuildNumber, Release/ReleaseName"
    & "&$expand=Build,Release"
)
```

### 2.3. Test Results (Detailed)

This table is crucial for granular analysis (Flakiness, specific failure reasons).

```powerquery
Source = OData.Feed("https://analytics.dev.azure.com/{Org}/{Project}/_odata/v3.0-preview/TestResults?"
    & "$select=TestResultId, TestRunId, TestCaseReferenceId, TestCaseTitle, Outcome, DurationSeconds, ErrorMessage, StackTrace, Priority, AutomatedTestName"
)
```

### 2.4. Pipeline Builds

```powerquery
Source = OData.Feed("https://analytics.dev.azure.com/{Org}/{Project}/_odata/v3.0-preview/Builds?"
    & "$select=BuildId, BuildNumber, Definition/Name, SourceBranch, StartTime, FinishTime, Result"
    & "&$expand=Definition"
)
```

## 3. Transformation Logic (Power Query M)

### 3.1. Flattened Test Results

To analyze test failures effectively, we create a "Flattened Test Results" table that combines Test Run metadata with detailed Test Results.

**M Script for `Flattened Test Results`:**

```powerquery
let
    // Load Test Runs
    SourceRuns = OData.Feed("https://analytics.dev.azure.com/{Org}/{Project}/_odata/v3.0-preview/TestRuns?$select=TestRunId,Title,StartedDate,CompletedDate,Build/BuildNumber,Release/ReleaseName&$expand=Build,Release"),

    // Load Test Results
    SourceResults = OData.Feed("https://analytics.dev.azure.com/{Org}/{Project}/_odata/v3.0-preview/TestResults?$select=TestResultId,TestRunId,TestCaseReferenceId,TestCaseTitle,Outcome,DurationSeconds,Priority,AutomatedTestName"),

    // Join Tables
    MergedQueries = Table.NestedJoin(SourceResults, {"TestRunId"}, SourceRuns, {"TestRunId"}, "TestRun", JoinKind.LeftOuter),

    // Expand TestRun Column
    ExpandedTestRun = Table.ExpandTableColumn(MergedQueries, "TestRun", {"Title", "StartedDate", "CompletedDate", "Build", "Release"}, {"RunTitle", "RunStartedDate", "RunCompletedDate", "Build", "Release"}),

    // Expand Build and Release
    ExpandedBuild = Table.ExpandRecordColumn(ExpandedTestRun, "Build", {"BuildNumber"}, {"BuildNumber"}),
    ExpandedRelease = Table.ExpandRecordColumn(ExpandedBuild, "Release", {"ReleaseName"}, {"ReleaseName"}),

    // Custom Column: Test Date (Date only)
    AddDate = Table.AddColumn(ExpandedRelease, "TestDate", each DateTime.Date([RunStartedDate]), type date),

    // Custom Column: Is Automated?
    AddIsAutomated = Table.AddColumn(AddDate, "IsAutomated", each if [AutomatedTestName] <> null then true else false, type logical)
in
    AddIsAutomated
```

### 3.2. Defect Leakage Logic

Create a calculated column in your `WorkItems` table (or via DAX) to determine if a bug leaked to production.

*   **Logic**: If `FoundInEnvironment` (custom field) is 'Production' or `Tags` contains 'Prod-Bug', then it is a Leak.
*   **Alternative**: Compare `CreatedDate` with `ReleaseDate`. If Created > Release, it's a leak.

## 4. Advanced Modelling: Star Schema

To enable traceability, construct a Star Schema:

*   **Fact_TestResults**: The "Flattened Test Results" table created above.
*   **Fact_WorkItems**: Contains User Stories and Bugs.
*   **Bridge_TestToWorkItem**: A bridge table linking Test Cases to Requirements (User Stories).
    *   *Source*: `WorkItemLinks` OData entity.
    *   *Filter*: `LinkTypeName eq 'Tested By'` (or similar).
*   **Dim_Date**: Standard date table.
*   **Dim_Area**: Area Paths.
*   **Dim_Build**: Build details.

**Relationships**:
*   `Fact_TestResults` (TestRunId) -> `Dim_Build` (BuildId)
*   `Fact_TestResults` (TestDate) -> `Dim_Date` (Date)
*   `Fact_WorkItems` (WorkItemId) <-> `Bridge_TestToWorkItem` (SourceWorkItemId)
*   `Bridge_TestToWorkItem` (TargetWorkItemId) <-> `Fact_TestResults` (TestCaseId - *requires mapping Test Case ID from Test Results*)

## 5. DAX Formulas

### 5.1. Pass Rate %

```dax
Pass Rate % =
VAR TotalTests = COUNTROWS('Fact_TestResults')
VAR PassedTests = CALCULATE(COUNTROWS('Fact_TestResults'), 'Fact_TestResults'[Outcome] = "Passed")
RETURN
    DIVIDE(PassedTests, TotalTests, 0)
```

### 5.2. Rolling 7-day Defect Trend

Shows the volume of bugs created in the last 7 days.

```dax
Rolling 7-day Defect Trend =
CALCULATE(
    COUNTROWS('Fact_WorkItems'),
    'Fact_WorkItems'[WorkItemType] = "Bug",
    DATESINPERIOD('Dim_Date'[Date], MAX('Dim_Date'[Date]), -7, DAY)
)
```

### 5.3. Defect Density

Number of bugs per Story Point delivered.

```dax
Defect Density =
VAR TotalBugs = CALCULATE(COUNTROWS('Fact_WorkItems'), 'Fact_WorkItems'[WorkItemType] = "Bug")
VAR TotalPoints = CALCULATE(SUM('Fact_WorkItems'[StoryPoints]), 'Fact_WorkItems'[WorkItemType] = "User Story", 'Fact_WorkItems'[State] = "Closed")
RETURN
    DIVIDE(TotalBugs, TotalPoints, 0)
```

### 5.4. Flakiness Index

Identifies tests that have both 'Passed' and 'Failed' outcomes within the same Build.

```dax
Flakiness Index =
VAR FlakyTests =
    FILTER(
        SUMMARIZE(
            'Fact_TestResults',
            'Fact_TestResults'[TestCaseTitle],
            'Fact_TestResults'[BuildNumber],
            "Outcomes", DISTINCTCOUNT('Fact_TestResults'[Outcome])
        ),
        [Outcomes] > 1
    )
RETURN
    COUNTROWS(FlakyTests)
```

### 5.5. MTTR (Mean Time to Repair)

Average days from Bug Creation to Closure.

```dax
MTTR (Days) =
VAR ClosedBugs = FILTER('Fact_WorkItems', 'Fact_WorkItems'[WorkItemType] = "Bug" && 'Fact_WorkItems'[State] = "Closed")
RETURN
    AVERAGEX(
        ClosedBugs,
        DATEDIFF('Fact_WorkItems'[CreatedDate], 'Fact_WorkItems'[ClosedDate], DAY)
    )
```

### 5.6. Test Execution Velocity

Rolling 7-day count of tests executed.

```dax
Test Execution Velocity =
CALCULATE(
    COUNTROWS('Fact_TestResults'),
    DATESINPERIOD('Dim_Date'[Date], MAX('Dim_Date'[Date]), -7, DAY)
)
```

## 6. Visualisation & Filtering

*   **Slicers**: Add slicers for `IterationPath`, `AreaPath`, and `ReleaseName` (from Test Runs).
*   **Traceability Matrix**: A Matrix visual with `User Story` on rows and `Test Outcome` on columns, showing the % coverage.
*   **Flakiness Report**: A Table visual listing `TestCaseTitle` and `BuildNumber` filtered by `Flakiness Index > 0`.
*   **Defect Trend**: Line chart with `Rolling 7-day Defect Trend` on the Y-axis and `Date` on the X-axis.

## 7. Conclusion

This solution provides a robust foundation for Quality Engineering analytics. By correlating Test Results with Requirements and Builds, you can move beyond simple pass/fail counts to measuring the true health and velocity of your delivery pipeline.
