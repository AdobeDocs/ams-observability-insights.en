---
title: Coverage, environments, and data retention
description: See what Observability Insights monitors in AEM Managed Services, how applications are represented, and how long monitoring data is retained.
feature: Operations
role: Admin
---

# Coverage, environments, and data retention {#coverage-environments-and-data-retention}

This page summarizes what data is collected in Observability Insights for AEM Managed Services and how that data is organized.

## Monitoring coverage {#monitoring-coverage}

Adobe monitors:

- AEM Author tiers with the Observability Insights APM Java plug-in
- AEM Publish tiers with the Observability Insights APM Java plug-in
- Hosted servers in the managed topology with the Observability Insights Infrastructure agent

Custom APM and infrastructure monitoring is enabled in both non-production and production Managed Services environments.

## How applications are represented {#how-applications-are-represented}

Each AEM Managed Services environment typically includes:

- One APM application for Author
- One APM application for Publish

All topologies in a Managed Services contract report into one Observability Insights account.

## Data retention {#data-retention}

APM metrics, infrastructure metrics, and related events are retained for up to **30 days**.

## Summary tables {#summary-tables}

| Coverage area  | What is monitored                          |
| -------------- | ------------------------------------------ |
| APM            | AEM Author and Publish applications        |
| Infrastructure | All hosted servers in the managed topology |

| Item                           | Representation                                                |
| ------------------------------ | ------------------------------------------------------------- |
| AEM environment                | One Author APM application and one Publish APM application    |
| Observability Insights account | One Adobe-managed account per Managed Services customer scope |

| Data type                         | Retention     |
| --------------------------------- | ------------- |
| APM metrics and events            | Up to 30 days |
| Infrastructure metrics and events | Up to 30 days |

## What this means operationally {#what-this-means-operationally}

- Observability Insights is suited to operational analysis, active incidents, and recent trend comparison.
- Historical analysis beyond the retention window should be handled through other reporting or archival processes if required.
- When investigating recurring issues, capture screenshots or exported evidence before the data ages out.
