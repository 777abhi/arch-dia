# Power BI Reporting Strategy for Azure DevOps using OData

This guide outlines a comprehensive strategy for building advanced Power BI reports using Azure DevOps (ADO) Analytics OData feeds. It focuses on extracting high-value performance metrics, including DORA metrics and team productivity indicators.

## 1. Introduction

Azure DevOps Analytics provides a rich OData feed that allows for deep, historical analysis of your development process. By connecting Power BI directly to these feeds, you can create custom reports that go beyond the built-in ADO dashboards, enabling you to measure Flow Efficiency, Cycle Time, MTTR, and more.

## 2. Prerequisites

*   **Power BI Desktop**: Installed on your machine.
*   **Access Permissions**: You must have **View Analytics** permission in Azure DevOps.
*   **Organization URL**: `https://dev.azure.com/{OrganizationName}`.

## 3. Data Modeling: The Star Schema

To effectively report on disparate modules (Boards, Repos, Pipelines), you should construct a **Star Schema** in Power BI. This model centers around Fact tables (transactional data) linked by shared Dimension tables.

### Fact Tables
*   **Fact_WorkItems**: Current state of all work items (from `WorkItems`).
*   **Fact_WorkItemHistory**: Daily snapshots of work items (from `WorkItemSnapshot`) for trend analysis.
*   **Fact_Builds**: History of pipeline runs (from `Builds` or `PipelineRuns`).
*   **Fact_PullRequests**: History of PRs (from `GitPullRequests` or `PullRequests` - check your `$metadata`).

### Dimension Tables
*   **Dim_Date**: A standard calendar table linked to all date fields (CreatedDate, ClosedDate, etc.).
*   **Dim_Project**: Project details (from `Projects`).
*   **Dim_Team**: Team details (from `Teams`).
*   **Dim_Person**: User details (from `Users`), linked to `AssignedTo`, `CreatedBy`, etc.

**Relationships**: Connect Dimensions to Facts using 1:* relationships.

## 4. Technical Approach & OData Queries

The base URL for ADO Analytics OData is:
`https://analytics.dev.azure.com/{Organization}/{Project}/_odata/v3.0-preview/`

> **Tip**: Always verify available entities by visiting `https://analytics.dev.azure.com/{Organization}/{Project}/_odata/v3.0-preview/$metadata` in your browser.

### 4.1. Azure Boards: Flow Efficiency & Cycle Time

**Goal**: Identify how much time work items spend in 'Active' vs 'Waiting' states.

**Query Strategy**:
Use `WorkItemBoardSnapshot` (if available) or `WorkItemSnapshot` to get the daily state of items.

**OData Query Example (Power Query M):**
```powerquery
Source = OData.Feed("https://analytics.dev.azure.com/{Org}/{Project}/_odata/v3.0-preview/WorkItemSnapshot?"
    & "$apply=filter(WorkItemType eq 'User Story' or WorkItemType eq 'Bug')"
    & "/groupby((DateValue, State, BoardColumn), aggregate($count as Count))"
)
```

**Metric Calculation (DAX):**
*   **Flow Efficiency %** = `(Active Time / (Active Time + Waiting Time)) * 100`
    *   *Note*: You must map your `BoardColumn` or `State` to "Active" (e.g., "In Progress") or "Waiting" (e.g., "Ready for Test", "Blocked").

### 4.2. Azure Repos: PR Pick-up Latency

**Goal**: Measure the time between PR creation and the first review action.

**Query Strategy**:
Use the `GitPullRequests` entity. If granular review timestamps aren't directly exposed in the main entity, you may need to use `creationDate` and `closedDate` as a proxy for total cycle time, or look for specific status change timestamps if available in your version of the API.

**OData Query Example:**
```powerquery
Source = OData.Feed("https://analytics.dev.azure.com/{Org}/{Project}/_odata/v3.0-preview/GitPullRequests?"
    & "$select=PullRequestId, CreationDate, ClosedDate, Status, SourceBranch, TargetBranch"
)
```

**Metric Calculation:**
*   **Pick-up Latency**: `DATEDIFF(CreationDate, FirstReviewDate, HOUR)`.
    *   *Workaround*: If `FirstReviewDate` is not available, track `CreationDate` vs `Active` status change if captured, or focus on **PR Cycle Time** (`ClosedDate - CreationDate`).

### 4.3. Azure Pipelines: Mean Time to Recovery (MTTR)

**Goal**: Track how quickly the team resolves broken builds/releases.

**Query Strategy**:
Use `Builds` or `PipelineRuns`. You need to identify a failure (Result = 'Failed') and the *next* success (Result = 'Succeeded') for the same Definition.

**OData Query Example:**
```powerquery
Source = OData.Feed("https://analytics.dev.azure.com/{Org}/{Project}/_odata/v3.0-preview/Builds?"
    & "$filter=result eq 'Succeeded' or result eq 'Failed'"
    & "&$select=BuildId, DefinitionId, FinishTime, Result"
    & "&$orderby=FinishTime desc"
)
```

**Metric Calculation (DAX/M):**
1.  Sort data by `DefinitionId` and `FinishTime`.
2.  Identify "Failure Blocks": A sequence of failures followed by a success.
3.  **MTTR** = `Average(SuccessTime - FirstFailureTime)`.

### 4.4. Azure Plans: Scope Creep

**Goal**: Measure work added to a sprint *after* it has started.

**Query Strategy**:
Use `WorkItemSnapshot` combined with Iteration data. Compare the list of items in an Iteration at the `StartDate` vs the `EndDate`.

**OData Query Example:**
```powerquery
Source = OData.Feed("https://analytics.dev.azure.com/{Org}/{Project}/_odata/v3.0-preview/WorkItemSnapshot?"
    & "$select=WorkItemId, DateValue, Iteration/StartDate, Iteration/EndDate"
    & "&$expand=Iteration"
)
```

**Metric Calculation:**
*   **Scope Creep**: Count of Work Items where `CreatedDate` > `Iteration.StartDate` AND `IterationPath` == Current Iteration.
    *   *Alternative*: Count of items where `DateValue` (snapshot date) is within the sprint, but the item was *not* present in the snapshot on `Iteration.StartDate`.

## 5. Visualization Strategy

| Metric | Recommended Visual | Why? |
| :--- | :--- | :--- |
| **Flow Efficiency** | **100% Stacked Bar Chart** | Shows the ratio of Active vs. Waiting time per sprint or month. |
| **Cycle Time** | **Control Chart (Scatter Plot)** | visualizes individual item completion times and outliers against the average. |
| **Scope Creep** | **Line & Clustered Column Chart** | Columns for "Planned Work", Line for "Added Work" (Scope Creep). |
| **MTTR / Failure Rate** | **Line Chart** | Trends over time are crucial for DORA metrics. |
| **PR Latency** | **Box & Whisker Plot** | Shows the distribution of wait times (median, quartiles) to identify consistency issues. |

## 6. Conclusion

By leveraging these OData entities and the Star Schema approach, you can build a robust engineering efficiency dashboard that provides actionable insights into your team's delivery performance.
