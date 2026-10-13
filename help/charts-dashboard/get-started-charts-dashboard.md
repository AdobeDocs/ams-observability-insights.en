---
title: Get started with charts and dashboards
description: Learn how to visualize Observability Insights data with charts and dashboards.
---

## Charts and dashboards: visualizing your data {#charts-and-dashboards-visualizing-your-data}

Queries find the information you need. Charts and dashboards make that information useful to your organization. This doc shows how SQL results become charts, and how to build, organize and share dashboards.

### Charts

Every SQL result can be rendered as one of six chart types.

| Chart type      | Use it for                                          | Query shape                                     |
| --------------- | --------------------------------------------------- | ----------------------------------------------- |
| **Line**        | Trends over time; one line per facet value          | Aggregate + `TIMESERIES`                        |
| **Area**        | Volume over time, such as throughput by service     | Aggregate + `TIMESERIES`                        |
| **Bar**         | Comparing categories; sparse time series            | Aggregate + `FACET` (± `TIMESERIES`)            |
| **Pie / Donut** | Share of a whole (status code mix, traffic by tier) | One aggregate + `FACET`, few values             |
| **Number**      | A single KPI, with optional colour thresholds       | Billboard (aggregates only)                     |
| **Table**       | Multiple aggregates per group, or event lists       | Several aggregates + `FACET`, or raw attributes |

After a widget is on a dashboard you can:

- **Edit** its query, title, chart type and unit (pencil icon).
- **Duplicate** it to create a variant, such as the same chart for another tier.
- **Download CSV** of the current rows, or **PNG** at 2× resolution.
- **Drag** it by the grip handle and **resize** it from the bottom-right corner.

### Dashboards

Charts work well on their own, but dashboards bring them together. A dashboard is a named, tenant-scoped collection of **tabs**. Each tab is a 12-column grid of **widgets**.

| Concept         | Meaning                                                                                                                         |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| **Dashboard**   | A named collection of 1–40 tabs, stored server-side for the selected tenant.                                                    |
| **Tab**         | One grid of widgets (up to 200). Rename it with the pencil, delete it with ×, add one with **+**. At least one tab must remain. |
| **Widget**      | A **single query**, a **formula**, or **text** (a note or a collapsible section header).                                        |
| **Variable**    | A picker above the grid. Its value is injected as `{{name}}`.                                                                   |
| **Time picker** | The window for every widget on the dashboard.                                                                                   |
