---
title: Metrics, coverage, and retention
description: Summary reference for what Synoptryx monitors in AEM Managed Services and how long monitoring data is retained.
feature: Operations
role: Admin
---

# Metrics, coverage, and retention {#metrics-coverage-and-retention}

Use this page as a quick reference for the scope of Synoptryx data in AEM Managed Services.

## Coverage summary {#coverage-summary}

| Coverage area  | What is monitored                      |
| -------------- | -------------------------------------- |
| APM            | AEM Author and Publish applications    |
| Infrastructure | Hosted servers in the managed topology |

## Application representation {#application-representation}

| Item              | Typical representation                                        |
| ----------------- | ------------------------------------------------------------- |
| AEM environment   | One Author APM application and one Publish APM application    |
| Synoptryx account | One Adobe-managed account per Managed Services customer scope |

## Data retention summary {#data-retention-summary}

| Data type                         | Retention     |
| --------------------------------- | ------------- |
| APM metrics and events            | Up to 30 days |
| Infrastructure metrics and events | Up to 30 days |

## Operational notes {#operational-notes}

- Data is intended for active operations and recent historical analysis.
- Teams should capture incident evidence within the retention window.
- More detailed retention or export guidance can be added here later if it becomes available.

## Related content {#related-content}

- [Coverage, environments, and data retention](../get-started/coverage-and-data.md)
- [APM dashboard reference](apm-dashboard-reference.md)
- [Infrastructure dashboard reference](infrastructure-dashboard-reference.md)
