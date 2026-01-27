# Building DORA Metrics with Azure DevOps Node.js API and TypeScript

This guide provides a step-by-step approach to calculating DORA (DevOps Research and Assessment) metrics using the Azure DevOps (ADO) Node.js API and a custom TypeScript utility.

## What are DORA Metrics?

DORA metrics are four key measures used to evaluate the performance of software development teams:

1.  **Deployment Frequency (DF):** How often an organization successfully releases to production.
2.  **Lead Time for Changes (LT):** The amount of time it takes a commit to get into production.
3.  **Change Failure Rate (CFR):** The percentage of deployments causing a failure in production.
4.  **Mean Time to Restore (MTTR):** How long it takes to recover from a failure in production.

## Prerequisites

Before starting, ensure you have:

*   **Node.js** installed (v14 or later recommended).
*   **TypeScript** installed globally or locally in your project.
*   Access to an **Azure DevOps Organization** and **Project**.
*   A **Personal Access Token (PAT)** with permissions to read Builds, Releases, and Work Items.

## Step 1: Project Setup

1.  Initialize a new Node.js project:

    ```bash
    mkdir dora-metrics-utility
    cd dora-metrics-utility
    npm init -y
    ```

2.  Install necessary dependencies:

    ```bash
    npm install azure-devops-node-api dayjs dotenv
    npm install typescript @types/node ts-node --save-dev
    ```

    *   `azure-devops-node-api`: The official ADO client for Node.js.
    *   `dayjs`: A lightweight library for date manipulation.
    *   `dotenv`: To manage environment variables (like your PAT).

3.  Initialize TypeScript configuration:

    ```bash
    npx tsc --init
    ```

4.  Create a `.env` file to store your credentials:

    ```env
    ADO_ORG_URL=https://dev.azure.com/your-organization
    ADO_PROJECT=your-project-name
    ADO_PAT=your-personal-access-token
    ```

## Step 2: Create the TypeScript Utility

Create a file named `dora-metrics.ts`. This script will connect to ADO, fetch the necessary data, and calculate the metrics.

### 2.1. Basic Setup and Connection

```typescript
import * as azdev from "azure-devops-node-api";
import * as ba from "azure-devops-node-api/BuildApi";
import { Build, BuildStatus, BuildResult } from "azure-devops-node-api/interfaces/BuildInterfaces";
import * as dayjs from "dayjs";
import * as dotenv from "dotenv";

dotenv.config();

const orgUrl = process.env.ADO_ORG_URL || "";
const token = process.env.ADO_PAT || "";
const project = process.env.ADO_PROJECT || "";

async function getBuildApi(): Promise<ba.IBuildApi> {
    const authHandler = azdev.getPersonalAccessTokenHandler(token);
    const connection = new azdev.WebApi(orgUrl, authHandler);
    return await connection.getBuildApi();
}
```

### 2.2. Fetching Deployments (Builds)

To calculate **Deployment Frequency** and **Change Failure Rate**, we need to fetch build history. We assume that a specific build definition (pipeline) represents a deployment to production.

```typescript
async function getDeployments(buildDefinitionId: number, days: number): Promise<Build[]> {
    const buildApi = await getBuildApi();
    const minTime = dayjs().subtract(days, 'day').toDate();

    const builds = await buildApi.getBuilds(
        project,
        [buildDefinitionId], // Filter by your production pipeline ID
        undefined,
        undefined,
        minTime,
        undefined,
        undefined,
        undefined,
        BuildStatus.Completed,
        undefined,
        undefined,
        undefined,
        undefined, // maxBuildsPerDefinition
        undefined, // queryOrder
        undefined, // reasonFilter
        undefined, // statusFilter
        undefined, // tagFilters
        undefined  // properties
    );

    return builds;
}
```

### 2.3. Calculating Deployment Frequency

Deployment Frequency is simply the count of successful deployments over a given period.

```typescript
function calculateDeploymentFrequency(builds: Build[], days: number): string {
    const successfulBuilds = builds.filter(b => b.result === BuildResult.Succeeded);
    const totalDeployments = successfulBuilds.length;

    // Frequency per day (or week, month)
    const frequencyPerDay = totalDeployments / days;

    return `${totalDeployments} deployments in ${days} days (${frequencyPerDay.toFixed(2)} per day)`;
}
```

### 2.4. Calculating Change Failure Rate

Change Failure Rate is the percentage of deployments that resulted in a failure.

```typescript
function calculateChangeFailureRate(builds: Build[]): string {
    const totalDeployments = builds.length;
    if (totalDeployments === 0) return "0%";

    // Assuming BuildResult.Failed indicates a failure.
    // You might also need to track rollbacks or specific tags depending on your process.
    const failedDeployments = builds.filter(b => b.result === BuildResult.Failed).length;

    const failureRate = (failedDeployments / totalDeployments) * 100;

    return `${failureRate.toFixed(2)}% (${failedDeployments} failures out of ${totalDeployments} deployments)`;
}
```

### 2.5. Calculating Lead Time for Changes

Lead Time requires comparing the time a commit happened vs. the time it was deployed.

*Note: This is a simplified version. For accurate results, you need to traverse the linked changesets or work items associated with each build.*

```typescript
async function calculateLeadTime(builds: Build[]): Promise<string> {
    const successfulBuilds = builds.filter(b => b.result === BuildResult.Succeeded);

    if (successfulBuilds.length === 0) return "N/A";

    let totalLeadTimeHours = 0;
    let count = 0;

    // This is a simplified example. In reality, you'd fetch the commit associated with the build
    // or the earliest commit in the batch of changes.
    // build.startTime is when the deployment started.
    // You would need to fetch the associated source version's timestamp.

    // For demonstration, we'll assume we can get a "commitTime" (pseudo-code)
    // You would use gitApi.getCommit(commitId) to get the actual time.

    for (const build of successfulBuilds) {
        const deployTime = dayjs(build.startTime);
        // Placeholder: Assuming the build was triggered immediately after a PR merge for simplicity
        // In a real scenario, fetch the commit time from the repository API using build.sourceVersion
        const commitTime = dayjs(build.queueTime);

        const leadTime = deployTime.diff(commitTime, 'hour');
        totalLeadTimeHours += leadTime;
        count++;
    }

    const avgLeadTime = totalLeadTimeHours / count;
    return `${avgLeadTime.toFixed(2)} hours`;
}
```

### 2.6. Calculating Mean Time to Restore (MTTR)

MTTR measures how long it takes to resolve an incident. This usually requires querying **Work Items** (e.g., Bugs or Incidents) rather than just builds.

```typescript
// You would need to import WorkItemTrackingApi
import * as wit from "azure-devops-node-api/WorkItemTrackingApi";

async function getIncidents(days: number): Promise<any[]> {
    const authHandler = azdev.getPersonalAccessTokenHandler(token);
    const connection = new azdev.WebApi(orgUrl, authHandler);
    const witApi = await connection.getWorkItemTrackingApi();

    const wiql = `
        SELECT [System.Id], [System.CreatedDate], [Microsoft.VSTS.Common.ClosedDate]
        FROM WorkItems
        WHERE [System.WorkItemType] = 'Incident'
        AND [System.CreatedDate] >= @Today - ${days}
        AND [System.State] = 'Closed'
    `;

    const result = await witApi.queryByWiql({ query: wiql });
    if (!result.workItems || result.workItems.length === 0) return [];

    const ids = result.workItems.map(wi => wi.id).filter((id): id is number => id !== undefined);
    const workItems = await witApi.getWorkItems(ids);

    return workItems;
}

function calculateMTTR(incidents: any[]): string {
    if (incidents.length === 0) return "N/A";

    let totalRestoreTimeHours = 0;

    for (const incident of incidents) {
        const created = dayjs(incident.fields['System.CreatedDate']);
        const closed = dayjs(incident.fields['Microsoft.VSTS.Common.ClosedDate']);

        totalRestoreTimeHours += closed.diff(created, 'hour');
    }

    const mttr = totalRestoreTimeHours / incidents.length;
    return `${mttr.toFixed(2)} hours`;
}
```

### 3. Main Execution Block

Combine everything into a main function to run the utility.

```typescript
async function run() {
    try {
        const buildDefinitionId = 123; // Replace with your actual Build Definition ID
        const daysToAnalyze = 30;

        console.log(`Analyzing metrics for the last ${daysToAnalyze} days...`);

        // 1. Fetch Builds
        const builds = await getDeployments(buildDefinitionId, daysToAnalyze);

        // 2. Deployment Frequency
        console.log(`Deployment Frequency: ${calculateDeploymentFrequency(builds, daysToAnalyze)}`);

        // 3. Change Failure Rate
        console.log(`Change Failure Rate: ${calculateChangeFailureRate(builds)}`);

        // 4. Lead Time for Changes
        console.log(`Lead Time for Changes: ${await calculateLeadTime(builds)}`);

        // 5. Mean Time to Restore (MTTR)
        // const incidents = await getIncidents(daysToAnalyze);
        // console.log(`MTTR: ${calculateMTTR(incidents)}`);

    } catch (error) {
        console.error("Error calculating metrics:", error);
    }
}

run();
```

## Step 3: Running the Utility

1.  Compile the TypeScript code:
    ```bash
    npx tsc
    ```

2.  Run the generated JavaScript file:
    ```bash
    node dora-metrics.js
    ```

    Or run directly with `ts-node`:
    ```bash
    npx ts-node dora-metrics.ts
    ```

## Conclusion

This utility provides a foundation for automating your DORA metrics collection. You can extend it by:
*   Refining the "Lead Time" calculation by tracing Git commits.
*   Improving "MTTR" accuracy by properly tagging production incidents.
*   Visualizing the data using a dashboard or exporting it to a database.
