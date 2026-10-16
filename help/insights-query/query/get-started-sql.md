---
title: Get started with SQL
description: Learn how to query Observability Insights telemetry using SQL.
---

# Get started

**Adobe Observability Insights Query Language (SQL)** — query, chart and dashboard your telemetry in Adobe Observability Insights.

|                |                                                                  |
| -------------- | ---------------------------------------------------------------- |
| **Applies to** | Insights UI → **Explore** and **Dashboards** (custom dashboards) |
| **Audience**   | Engineers, SREs and power users who know SQL                     |

>[!TIP]
>
> Every code block marked `sql` is a SQL query that you can paste straight into **Explore → SQL** or into **Dashboards → Add widget → SQL**.

## Get started with SQL: the language of your telemetry {#get-started-with-sql-the-language-of-your-telemetry}

Insights Query language is a SQL-like query language for your Observability Insights data. It is the same language behind every custom dashboard widget and the **Explore** page, so you can build the views your team needs.

SQL works with **entities**, which are friendly logical views such as `Transaction`, `Span` or `CpuUtilization`. You never query storage tables directly. Observability Insights compiles each SQL query into a safe, parameterised warehouse query. It always scopes that query to your tenant and the selected time window.

```sql
SELECT count(*) AS requests FROM Transaction TIMESERIES AUTO
```

### What you can do with SQL

Use SQL to:

- **Build charts** of throughput, error rate, latency, Apdex, CPU, memory, JVM heap, disk and network over time.
- **Answer specific questions**, such as "which URL paths on the publish tier have the worst p95?" or "which downstream domain is failing?"
- **Investigate incidents** by drilling from a KPI down to individual requests and trace IDs.
- **Build dashboards** with KPIs, trends, tables, formulas, notes, section headers and variables.
- **Profile AEM workloads** (Adobe Experience Manager) using Sling, JSP, JCR, dispatcher and GraphQL attributes.

SQL supports both **event queries**, which return individual request rows, and **aggregate queries**, which return totals, percentiles, rates, breakdowns and time series.

>[!TIP]
>
> The templates in **Dashboards → Create dashboard** are built entirely from SQL. Open any template widget with the pencil icon to see, and learn from, its query.

### What you can query

SQL can query four families of data. The complete list is in the [Data dictionary](/help/references/data-dictionary.md).

| Family                               | Entities                                                                                                                                                           | Typical questions                                                    |
| ------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------- |
| **Traces (APM)**                     | `Transaction` (inbound requests), `AppRequest` (requests + message consumers), `Span` (every span)                                                                 | Throughput, latency, errors, Apdex, downstream calls, AEM code paths |
| **Curated host and runtime metrics** | `CpuUtilization`, `MemoryUtilization`, `LoadAverage5m`, `DiskUtilization`, `NetworkReceiveRate`, `JvmHeapUsed`, `JvmThreadCount`, `DispatcherRequestRate` and more | Saturation, capacity, JVM health, dispatcher load                    |
| **Generic metrics**                  | `MetricGauge`, `MetricSum`                                                                                                                                         | Any OpenTelemetry metric by name, metric discovery                   |
| **Pre-aggregated span composition**  | `SpanBreakdown`                                                                                                                                                    | Which child operations make up a transaction                         |

>[!NOTE]
>
> Logs, alert definitions and inventory relationships aren't available.

### Where to use SQL

| Where                   | How to open it                                                                 | What it's for                                                                                                                                                                        |
| ----------------------- | ------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Explore**             | Left navigation → **Explore**                                                  | Ad hoc analysis. Use the **Visual Builder** or the **SQL** tab, **Validate**, then **Run** (⌘/Ctrl + Enter). Includes **History** and **Saved** queries and a **Query guide** panel. |
| **Dashboards**          | Left navigation → **Dashboards** → open or create a dashboard → **Add widget** | Persistent charts, KPIs and tables, combined into dashboards.                                                                                                                        |
| **Formula widgets**     | **Add widget → Formula**                                                       | Combine 2–6 SQL queries with arithmetic (ratios, headroom, burn rates).                                                                                                              |
| **Dashboard variables** | **+ Variable → From a query**                                                  | Populate a picker from the distinct values of a SQL `FACET` query.                                                                                                                   |
| **Query inspection**    | Hover a chart → inspect query                                                  | See the SQL behind a built-in or dashboard chart.                                                                                                                                    |

### SQL query structure

SQL follows a clause-based syntax. This query counts all inbound requests in the selected window:

```sql
SELECT count(*) FROM Transaction
```

Here are all the available clauses:

```text
SELECT  function(attribute) [FILTER (WHERE condition)] [AS alias] [, ...]
        | attribute [AS alias] [, ...]
FROM    Entity
  [WHERE      condition [AND | OR ...]]
  [FACET      attribute [, attribute ...]]        -- up to 5
  [SINCE      n unit [AGO]]
  [UNTIL      NOW | n unit [AGO]]
  [TIMESERIES AUTO | MAX | n unit]
  [ORDER BY   alias [ASC | DESC]]
  [LIMIT      number]                              -- 1 to 5000
```

Here are the rules SQL follows:

| Rule                         | Details                                                                                                                                                                                                   |
| ---------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Required clauses**         | `SELECT` and `FROM` are required. Every other clause is optional. You can start with either: `FROM Transaction SELECT count(*)` is also valid.                                                            |
| **Clause order**             | Clauses after `FROM` can appear in any order. Each clause may appear only once.                                                                                                                           |
| **Query size**               | A query must be 4 KB or less.                                                                                                                                                                             |
| **Case sensitivity**         | Keywords, function names, entity names and attribute names are **not** case sensitive. String values **are** case sensitive (`'Error'` ≠ `'error'`).                                                      |
| **Strings**                  | Use single quotes: `WHERE status = 'Error'`. To include a quote, double it: `'O''Brien'`. Double quotes aren't accepted.                                                                                  |
| **Literals**                 | Strings, numbers (including negative and decimal), `TRUE`, `FALSE`, `NULL`.                                                                                                                               |
| **Attribute names**          | Names can contain letters, digits, `_` and `.` (for example `http.statusCode`). No backticks are needed.                                                                                                  |
| **Aggregates vs attributes** | A query either selects raw attributes (an event list) or aggregates, never both. To group, use `FACET`.                                                                                                   |
| **Time window**              | The default is the last hour. The maximum is 31 days. On dashboards and in Explore, the **time picker** sets the window (see [Time window](how-to-query-sql.md#time-window-precedence)).                  |
| **Not supported**            | Semicolons, comments, subqueries, joins, `SELECT *`, math between attributes in `SELECT`, `HAVING`, `COMPARE WITH`. See [What SQL doesn't do](sql-query-examples.md#what-sql-doesnt-do) for alternatives. |
