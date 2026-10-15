---
title: SQL query examples
description: Explore SQL query examples for Observability Insights telemetry.
---

## SQL query examples

Here's a more complete example. For this query:

- You choose `Transaction` as the entity.
- You use `SELECT` to calculate the 95th percentile duration.
- You group the results by service with `FACET`.
- You use `TIMESERIES AUTO` to plot the data over an automatically sized interval.

```sql
FROM Transaction SELECT percentile(duration, 95) AS p95 FACET service TIMESERIES AUTO
```

This produces one line per service on a Line chart, with the 10 busiest services shown by default.

### Basic query of inbound requests

+++View query

Average response time, in milliseconds, of all inbound requests:

```sql
SELECT average(duration) FROM Transaction
```

+++

### Attribute names that contain dots

+++View query

Dotted names don't need quoting:

```sql
SELECT count(*) FROM Transaction WHERE http.statusCode >= 500 FACET http.route
```

+++

### Query returning multiple columns

+++View query

Several aggregates in one query become columns in a Table:

```sql
SELECT count(*) AS requests, average(duration) AS avgMs, percentile(duration, 95) AS p95, max(duration) AS maxMs FROM Transaction FACET service
```

+++

### Conditional aggregates in one pass

+++View query

Total, errors and 5xx side by side:

```sql
SELECT count(*) AS total, count(*) FILTER (WHERE status = 'Error') AS errors, count(*) FILTER (WHERE http.statusCode >= 500) AS http5xx FROM Transaction FACET service
```

+++

### Querying a curated metric

+++View query

Average CPU per host over time:

```sql
SELECT average(value) AS cpu FROM CpuUtilization FACET host TIMESERIES AUTO
```

+++

### Querying any OpenTelemetry metric by name

+++View query

Use `MetricGauge` / `MetricSum` for metrics without a curated entity:

```sql
SELECT latest(value) AS bytesUsed FROM MetricSum WHERE metric = 'jvm.memory.used' FACET host, pool
```

+++

<a id="what-sql-doesnt-do"></a>

## What SQL doesn't do

SQL is deliberately small. That keeps queries safe, fast and tenant-isolated. When you need one of these capabilities, use the alternative.

| Capability                                  | Status        | Alternative                                                                                                                            |
| ------------------------------------------- | ------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| `SELECT *`                                  | Not supported | List the attributes you want: `SELECT timestamp, name, duration, traceId …`.                                                           |
| Mixing raw attributes and aggregates        | Not supported | Use `FACET attr`, or `latest(attr)` / `earliest(attr)`.                                                                                |
| Math in `SELECT` (`a - b`, `count(*) / 60`) | Not supported | **Formula** widget (dashboards).                                                                                                       |
| Joins and subqueries                        | Not supported | **Formula** widget, or two widgets side by side.                                                                                       |
| `HAVING`                                    | Not supported | `ORDER BY alias` + `LIMIT`.                                                                                                            |
| `COMPARE WITH`                              | Not supported | Change the time picker, or put two widgets on different tabs.                                                                          |
| `FACET CASES`                               | Not supported | One `count(*) FILTER (WHERE …)` column per case (see [Tutorial 3](../../tutorials/tutorial-3.md#group-values-into-cases-with-filter)). |
| `histogram()`                               | Not supported | Bucketed `FILTER` columns (see [Tutorial 3](../../tutorials/tutorial-3.md#build-a-latency-histogram)).                                 |
| `uniques()` / `keyset()`                    | Not supported | `FACET attr LIMIT n` lists distinct values. Discovery queries list metrics and populated attributes.                                   |
| `RLIKE` (regex)                             | Not supported | `LIKE` / `NOT LIKE` with `%` and `_` wildcards, or `IN (…)`.                                                                           |
| Comments (`--`, `//`, `/* */`)              | Not supported | Put notes in the widget title or a **Text** widget.                                                                                    |
| Absolute dates in `SINCE`                   | Not supported | Use the time picker's absolute range.                                                                                                  |
| Type conversion (`numeric()`, `string()`)   | Not needed    | Attributes are typed in the catalog.                                                                                                   |
