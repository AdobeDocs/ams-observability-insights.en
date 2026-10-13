---
title: Create a dashboard
description: Learn how to create, configure, save, and share Observability Insights dashboards.
---

## Create a dashboard, step by step

### Step 1: Before you start

- **Select a single customer** in the tenant switcher. **Create dashboard** is disabled while **All accounts** is selected. SQL can't run against the demo tenant.
- **Roles.** Anyone with dashboard access can add, edit and save widgets. Creating or editing **variables**, and inspecting a widget's query, is limited to **org admins** and **super admins**.

### Step 2: Create the dashboard

1. Open **Dashboards** in the left navigation.
2. Click **Create dashboard**.
3. Choose a starting point:

   | Option                       | What you get                                                                                             |
   | ---------------------------- | -------------------------------------------------------------------------------------------------------- |
   | **Build a custom dashboard** | An empty dashboard with one tab.                                                                         |
   | **CPU Basics**               | Host and process CPU (user/system/iowait/steal), throughput, load, Apdex.                                |
   | **All-in-One Overview**      | KPIs (volume, error %, Apdex, p95), error/traffic pies, CPU/memory tables, JVM threads, slow operations. |
   | **Top Transactions**         | Apdex, p95 by operation, per-host load/CPU, slowest transactions with trace IDs, HTTP error breakdown.   |

4. Enter a **name** and confirm. The dashboard canvas opens.

### Step 3: Get to know the canvas

| Area                         | Controls                                                                                                  |
| ---------------------------- | --------------------------------------------------------------------------------------------------------- |
| **Top bar**                  | **Add widget**, **Guide** (in-app cheat sheet), **Import**, **Export**, **Save**, time picker and refresh |
| **Variable bar**             | Tenant badge, one picker per variable, **+ Variable** (admins)                                            |
| **Tab bar**                  | Switch tabs; rename (pencil), delete (×), add (+)                                                         |
| **Grid**                     | 12 columns. Drag widgets by the grip, resize from the corner (minimum 2×2).                               |
| **Widget header** (on hover) | Download CSV, Download PNG, Edit, Duplicate, Remove, drag handle                                          |

>[!IMPORTANT]
>
> **Saving is explicit.** Layout changes, new widgets, tabs and variables stay in your browser until you click **Save**. The button shows _Saving… → Saved_, or _Save failed_ (see [Troubleshooting](#troubleshooting)).

### Step 4: Add a widget

Click **Add widget**. The dialog has three modes: **Single query**, **Formula** and **Text**. It also has its own time picker, so you can preview the widget over any window.

**Option A: the Builder (no code)**

1. _(Optional)_ **Start with a helper**: choose one of 57 presets, grouped as Golden signals, APM, AEM, Transactions, Traces & spans, Infrastructure, Publishers, JVM & runtime and Data discovery. A preset fills in the title, query, chart type and unit.
2. **Show**: choose `Count`, `Average`, `Percentile` (and its level), `Sum`, `Min`, `Max` or `Unique count`, then an attribute.
3. **from**: choose the entity.
4. **Filter**: add rows of `attribute operator value` (`=`, `!=`, `>`, `<`, `>=`, `<=`, `LIKE`). All rows are combined with `AND`.
5. **Group by**: choose one facet attribute, and **top N** to keep only the busiest series.
6. **Over time**: turns on `TIMESERIES`, with an **every N minutes** bucket size.
7. Check the generated SQL under the form, then click **Run query**.

**Option B: the SQL editor**

1. Switch the toggle from **Builder** to **SQL**. Editing an existing widget always opens in SQL.
2. Type or paste a query. The editor autocompletes from the live catalog.
3. Use the **Query guide** panel to browse entities, attributes and examples.
4. Click **Run query**. You'll see _`<Entity>` query is valid_, a live preview and a summary (`N rows · N time points · N groups`). A _Sparse series_ warning appears when there are fewer than three time points.

>[!TIP]
>
> The Builder covers the most common queries. Switch to SQL for `apdex`, `percentage`, `rate`, `FILTER`, multiple aggregates, `OR`, `IN`, several facets or `ORDER BY`.

### Step 5: Configure how it looks

| Field                 | Notes                                                                                                                                                                   |
| --------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Title**             | Up to 200 characters. Can contain `{{variable}}` tokens.                                                                                                                |
| **Chart type**        | `Line`, `Area`, `Bar`, `Pie / Donut`, `Number`, `Table`. A chart type is recommended after each run until you choose one yourself.                                      |
| **Unit**              | Optional; up to 40 characters (`ms`, `%`, `req/s`, `MB`, `bytes`). Leave it blank to use the inferred unit (for example `ms` for `duration`, `%` for `CpuUtilization`). |
| **Colour thresholds** | Number widgets only. Up to 10 bands of the form _at or above `value` → Green / Amber / Red_. The highest matching band wins.                                            |

### Step 6: Save the widget, then the dashboard

**Save widget** is enabled only after a run returns data. New widgets are 6 columns × 3 rows and are packed two per row. Arrange them, then click **Save** in the top bar.

## Choose the right chart

| Query shape                                       | Best chart          | Why                                            |
| ------------------------------------------------- | ------------------- | ---------------------------------------------- |
| One aggregate, no `FACET`, no `TIMESERIES`        | **Number**          | A single KPI tile; add thresholds.             |
| Aggregate + `TIMESERIES`                          | **Line** / **Area** | A trend; one series per facet value.           |
| Aggregate + `FACET` (few values)                  | **Bar** / **Pie**   | Compares categories; pie for share of a whole. |
| Several aggregates + `FACET`                      | **Table**           | Each aggregate becomes a column.               |
| Raw attributes                                    | **Table**           | An event list.                                 |
| Aggregate + `FACET` + `TIMESERIES` on sparse data | **Bar**             | Reads better than gappy lines.                 |

When a Number widget's single row carries a text value (for example `latest(host)`), the widget shows it as a subtitle.

## Text widgets: notes and section headers

**Add widget → Text**:

- **Note**: plain text (up to 4,000 characters, blank lines preserved). Use it for runbook links, owners and SLO definitions. `{{variables}}` are rendered.
- **Section header**: tick _Make this a section header_. It spans the full width, and its chevron collapses every widget below it, up to the next section header. Use it for groupings such as **PUBLISH**, **AUTHOR** and **DISPATCHER**.

>[!TIP]
>
> Widget titles ending in `- PUBLISH` or `- AUTHOR` get a subtle colour cue, which makes AEM dashboards easier to scan by tier.

## The time picker

- It sets the window for **every** widget, and for query-driven variable options.
- It overrides `SINCE` / `UNTIL` in widget queries.
- The maximum window is **31 days**.
- `TIMESERIES AUTO` adapts its bucket size to the window.
- Each widget's subtitle shows the window actually applied, for example _Since 1 hour ago_.
- Results are cached for 30 seconds. Use the refresh button to force a re-run.

## Variables

Variables let one dashboard serve many contexts, such as one service, environment or threshold at a time. They are covered in depth in [Tutorial 3](#87-make-dashboards-reusable-with-variables).

>[!WARNING]
>
> **Known limitation in the current build:** `{{variables}}` work in widget **titles** and **text widgets**. Inside widget **SQL**, they render on the dashboard, but the **Add widget** preview and the server-side **Save** check reject the raw `{{name}}` token with `Unexpected character '{'`. Until this is fixed, use literal values in widget queries and use **Duplicate** to make variants.

## Formula widgets

A formula combines 2–6 named SQL queries arithmetically, for example `(errors / total) * 100`. Use one when a KPI needs more than one query, such as a ratio, headroom, a sum of rates or a burn rate. See [Tutorial 3](#86-combine-queries-with-formula-widgets).

## Share, export and import

| Action        | What it does                                                                                                                                                              |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Save**      | Persists the name, tabs, widgets, layout and variables. Every widget query is re-validated server-side; one invalid widget fails the whole save.                          |
| **Export**    | Downloads `<dashboard-name>.json` (tabs, widgets, variables).                                                                                                             |
| **Import**    | Loads a JSON export **into the dashboard that's open**, replacing its tabs and variables. Then click **Save**. Tip: create an empty dashboard first, then import into it. |
| **CSV / PNG** | Per-widget downloads from the widget header.                                                                                                                              |

>[!TIP]
>
> Export and Import let you version-control dashboards in Git, and promote a dashboard from one customer to another. The file format is described in the [Appendix](#142-export-file-format).

## Tips for dashboard and chart performance

- **Use tables for large result sets** instead of bars or many-series lines.
- **Split busy dashboards into tabs.** Only the visible tab's widgets run.
- **Always `LIMIT` faceted queries**, especially on high-cardinality attributes such as `urlPath`, `traceId` and `jcr.path`.
- **Filter early** on low-cardinality attributes (`service`, `kind`, `status`).
- **Prefer `TIMESERIES AUTO`** to fixed small buckets, and avoid `TIMESERIES MAX` except over a few minutes.
- **Design long-range trends for rollups.** See [Performance](#performance).
