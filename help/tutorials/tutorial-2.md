---
title: "Tutorial 2: Control your data"
description: Learn how to filter, group, and control query results in Observability Insights.
---

## Tutorial 2: Control your data {#tutorial-2-control-your-data}

In this tutorial you'll learn the functions and clauses that turn raw counts into answers: distinct counts, latest values, percentages, percentiles, wildcards, sorting and multi-dimensional grouping.

### Count distinct values with `uniqueCount`

How many hosts and distinct traces does each service have?

```sql
SELECT uniqueCount(host) AS hosts, uniqueCount(traceId) AS traces FROM Transaction FACET service
```

`uniqueCount` is exact, not an estimate.

To **list** the distinct values themselves (what NRQL calls `uniques()`), facet on the attribute:

```sql
SELECT count(*) AS requests FROM Transaction FACET host LIMIT 100
```

### Find the latest and earliest values

`latest(attr)` returns the most recent value, and `earliest(attr)` the oldest. They're useful for "current state" tables:

```sql
SELECT latest(value) AS currentCpu FROM CpuUtilization FACET host
```

```sql
SELECT earliest(timestamp) AS firstSeen, latest(timestamp) AS lastSeen FROM Transaction FACET service
```

### Calculate percentages

`percentage(*, WHERE condition)` returns the share of rows that match a condition. This is the error rate:

```sql
SELECT percentage(*, WHERE status = 'Error') AS errorPct FROM Transaction
```

Trend it per service:

```sql
SELECT percentage(*, WHERE status = 'Error') AS errorPct FROM Transaction FACET service TIMESERIES AUTO LIMIT 10
```

And track the share of slow requests:

```sql
SELECT percentage(*, WHERE duration > 2000) AS slowPct FROM Transaction FACET service
```

### Understand latency with `percentile`

Averages hide outliers. Percentiles show what most users experience (p50) and what the unlucky ones experience (p95, p99):

```sql
SELECT percentile(duration, 50) AS p50, percentile(duration, 95) AS p95, percentile(duration, 99) AS p99 FROM Transaction TIMESERIES AUTO
```

>[!TIP]
>
> p50, p95 and p99 of `Transaction.duration` are served from rollups over long windows, so they stay fast for weekly views.

### Make results readable with labels

Use `AS` with single quotes for labels that contain spaces:

```sql
SELECT count(*) AS 'Requests', percentile(duration, 95) AS 'p95 (ms)' FROM Transaction FACET service
```

>[!NOTE]
>
> Aliases apply only to `SELECT` items. Facet columns are always labelled with the attribute name.

### Match patterns and lists

`LIKE` uses `%` for any number of characters and `_` for exactly one:

```sql
SELECT count(*) AS requests FROM Transaction WHERE urlPath LIKE '/content/%.html' FACET urlPath LIMIT 20
```

Exclude static assets with `NOT LIKE`:

```sql
SELECT count(*) AS requests FROM Transaction WHERE urlPath NOT LIKE '%.js' AND urlPath NOT LIKE '%.css' AND urlPath NOT LIKE '%.png' FACET urlPath LIMIT 20
```

Match against a list with `IN` and `NOT IN` (up to 100 values):

```sql
SELECT count(*) AS requests FROM Transaction WHERE http.method IN ('POST', 'PUT', 'DELETE') FACET service
```

>[!NOTE]
>
> SQL has no regular expressions (`RLIKE`). Combine `LIKE` patterns with `OR`, or use `IN`.

### Sort and rank with `ORDER BY`

By default, faceted results are sorted by the first aggregate. Use `ORDER BY` to rank by something else, such as the slowest operations by p95:

```sql
SELECT count(*) AS requests, percentile(duration, 95) AS p95 FROM Transaction FACET name ORDER BY p95 DESC LIMIT 20
```

Or the least-used services first:

```sql
SELECT count(*) AS requests FROM Transaction FACET service ORDER BY requests ASC LIMIT 10
```

> **Tip**
>
> There's no `HAVING`. To find only "busy" operations, rank by `requests` and keep the top N, or add a `WHERE` that removes noise (for example `WHERE http.method != 'OPTIONS'`).

### Group by several attributes

`FACET` accepts up to five attributes. Each combination becomes one row or series:

```sql
SELECT count(*) AS requests FROM Transaction WHERE http.statusCode >= 400 FACET service, http.method, http.statusCode LIMIT 30
```

### Look at a past window

There's no `COMPARE WITH`. On dashboards and in Explore, change the **time picker**. When you call the API without an explicit range, `SINCE … UNTIL …` gives you any past window:

```sql
SELECT count(*) AS lastWeekSameHour FROM Transaction SINCE 169 hours ago UNTIL 168 hours ago
```
