---
title: Application Performance Monitoring (APM) with Synoptryx
description: Learn when to use the Synoptryx APM experience, what questions it answers, and where to find detailed dashboard metric definitions.
feature: Operations
role: Admin
---

# Application Performance Monitoring (APM) with Synoptryx {#application-performance-monitoring}

Synoptryx APM is the primary place to investigate application behavior in AEM Managed Services. It helps you understand request throughput, latency, error patterns, JVM health, and transaction traces across Author and Publish tiers.

Use this section when you need to answer questions such as:

- Is the issue isolated to Author or Publish?
- Did latency increase before the incident, or only after error rates changed?
- Which endpoints or transactions contribute most to traffic and slowdowns?
- Do traces show repository calls, external dependencies, or JVM pressure that explain the issue?

## What APM covers {#what-apm-covers}

AEM runs as a Java application on Jetty with Apache Felix OSGi modules, built on Apache Sling and Jackrabbit Oak. Adobe Managed Services, AEM Engineering, and Synoptryx Engineering jointly developed custom instrumentation for Managed Services environments.

That instrumentation gives you:

- **Meaningful transaction naming** — Sling extensions align transaction names with page structure and add a `requestURL` attribute on Insights events so you can correlate Sling URLs across dashboards.

![Synoptryx APM trace view showing a descriptive AEM transaction name with Sling health check route and span timeline](assets/image19a.png)

- **JCR instrumentation** — Repository-level operations (including XPath and JCR-SQL2) are categorized and attached to transaction traces in the database section of APM.

![Synoptryx APM trace view showing nested AEM component spans and execution timeline for a page request](assets/image19.png)

## How to use this guide {#how-to-use-this-guide}

Start with the investigation guidance on this page if you are actively diagnosing an issue, then use the reference when you need detailed metric or panel definitions.

- [APM dashboard reference](reference/apm-dashboard-reference.md) documents each dashboard section and metric.
- [Coverage, environments, and data retention](get-started/coverage-and-data.md) summarizes what data is collected and how long it is retained.

## Author and Publish applications {#author-and-publish-applications}

Author and Publish share a codebase but are monitored as **separate APM applications** so you can analyze each tier independently.

Every Managed Services environment includes:

- One APM application for Author
- One APM application for Publish

Select an application name in Synoptryx APM to open its overview and monitoring dashboard.

![Synoptryx APM application list showing Author and Publish applications](assets/image1a.png)

## Typical investigation workflow {#typical-investigation-workflow}

Use the following sequence for most incidents:

1. Confirm whether traffic, error rate, or latency changed materially.
2. Determine whether the behavior affects Author, Publish, or both.
3. Identify the endpoints, transactions, or status groups contributing most to the issue.
4. Open traces to inspect repository operations, downstream services, and execution timing.
5. Correlate APM findings with host metrics if the issue might be capacity-related.

## Dashboard sections {#dashboard-sections}

The APM dashboard includes these major sections:

- Overview
- RED Metrics (Rate • Errors • Duration)
- Traffic
- Latency & Performance
- Error Details
- Top Transactions
- JVM Health
- JVM Memory
- Garbage Collection

## What to review first {#what-to-review-first}

Use these dashboard areas as your first stop:

- **Overview** for total requests, throughput, error rate, and APDEX.
- **RED Metrics** to confirm whether the primary shift is volume, failure rate, or latency.
- **Traffic** to isolate endpoints or status code bands.
- **Latency & Performance** to compare current behavior against recent history.
- **Error Details** to determine whether failures are rising consistently or only in short spikes.

## Detailed dashboard reference {#detailed-dashboard-reference}

For panel-by-panel descriptions, metric names, screenshots, and units, see [APM dashboard reference](reference/apm-dashboard-reference.md).

## Questions to answer during investigation {#questions-to-answer}

- Did throughput change before the issue, or only after symptoms began?
- Are failures concentrated in one HTTP status band or one endpoint family?
- Is latency elevated broadly, or only for specific transactions?
- Do traces point to repository operations, downstream systems, or application code hot spots?

## Evidence to capture when escalating {#evidence-to-capture}

Capture these items when escalating or collaborating with Adobe Managed Services:

- Environment name and time window
- Whether Author, Publish, or both are affected
- Screenshots of overview, RED metrics, and error or latency charts
- Example trace IDs or transaction names
- Any correlated infrastructure anomalies

## Related content {#related-content}

- [Infrastructure monitoring](infrastructure-monitoring.md)
- [APM dashboard reference](reference/apm-dashboard-reference.md)
- [Coverage, environments, and data retention](get-started/coverage-and-data.md)
