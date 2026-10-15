---
title: "Tutorial 4: Advanced functions"
description: Learn how to use advanced SQL functions to analyze Observability Insights telemetry.
---

## Tutorial 4: Advanced functions {#tutorial-4-advanced-functions}

This tutorial covers the functions and entities that power deeper investigations: variability, growth, "worst example" lookups, share of total, transaction composition and arbitrary OpenTelemetry metrics.

### Measure consistency with `stddev`

Two operations can share an average and still behave very differently. The standard deviation shows how much latency varies:

```sql
SELECT count(*) AS requests, average(duration) AS avgMs, stddev(duration) AS stddevMs FROM Transaction FACET name ORDER BY stddevMs DESC LIMIT 20
```

A standard deviation larger than the average points to an operation with occasional very slow requests. Look at its p99 next.

### Spot growth with `derivative`

`derivative(attr)` returns the per-second change of the attribute's average between buckets. It's useful for spotting a gauge that's climbing, such as heap usage before a leak:

```sql
SELECT derivative(value) AS heapMBPerSec FROM JvmHeapUsed FACET host TIMESERIES 5 minutes
```

A sustained positive line means growth. Negative values mean the gauge is falling (for example, after garbage collection). Like `rate`, `derivative` needs `TIMESERIES` and must be the only aggregate.

### Find the worst example with ranked `latest`

The second argument of `latest` and `earliest` changes "most recent" to "highest by this attribute". `latest(traceId, duration)` returns the trace ID of the slowest request, so you can paste it into trace search:

```sql
SELECT count(*) AS requests, max(duration) AS slowestMs, latest(traceId, duration) AS slowestTrace, latest(urlPath, duration) AS slowestPath FROM Transaction FACET name ORDER BY slowestMs DESC LIMIT 15
```

Use `earliest(traceId, duration)` to fetch the fastest example for comparison.

### Find where time goes with `percentOfTotal`

`percentOfTotal(attr)` shows each group's share of the total. Applied to `duration`, it shows which operations consume most server time. This is often more useful than which operations are slowest:

```sql
SELECT count(*) AS requests, sum(duration) AS totalMs, percentOfTotal(duration) AS shareOfServerTime FROM Transaction FACET name LIMIT 15
```

`percentOfTotal` can't be combined with `TIMESERIES`.

### Break down transactions with `SpanBreakdown`

`SpanBreakdown` is pre-aggregated at ingest. It records which child operations (`segment`) each transaction (`rootOperation`) spends time in, minute by minute. It answers "what is this endpoint doing?" cheaply:

```sql
SELECT sum(callCount) AS calls, average(avgDurationMs) AS avgMs FROM SpanBreakdown WHERE rootOperation LIKE '%GET%' FACET rootOperation, segment LIMIT 25
```

Trend one segment over time:

```sql
SELECT average(avgDurationMs) AS avgMs FROM SpanBreakdown FACET segment TIMESERIES AUTO LIMIT 8
```

> [!NOTE]
>
> `avgDurationMs` isn't auto-tagged with a unit. Set the widget **Unit** to `ms`.

### Discover and query any metric

SQL has no `keyset()` or `SHOW EVENT TYPES`. Instead, list every metric that's reporting, together with its unit and latest value:

```sql
SELECT count(*) AS samples, latest(value) AS latestValue FROM MetricGauge FACET metric, unit LIMIT 200
```

```sql
SELECT count(*) AS samples, latest(value) AS latestValue FROM MetricSum FACET metric, unit LIMIT 200
```

Then query any metric by name. For example, JVM memory used, per pool:

```sql
SELECT latest(value) AS bytesUsed FROM MetricSum WHERE metric = 'jvm.memory.used' FACET host, pool LIMIT 50
```

To see which **span attributes** are populated for each service, count non-blank values:

```sql
SELECT count(*) FILTER (WHERE http.route != '') AS withRoute, count(*) FILTER (WHERE db.system != '') AS dbCalls, count(*) FILTER (WHERE peer.service != '') AS withPeer, count(*) FILTER (WHERE error.type != '') AS withErrorType FROM Span FACET service LIMIT 50
```

> [!TIP]
>
> When a curated entity exists (for example `CpuUtilization`), prefer it over `MetricGauge`. It applies the right filter and unit conversion, and it may be rollup-accelerated.

### Choose the right trace entity

| Use           | When you want                                                                               | Example                           |
| ------------- | ------------------------------------------------------------------------------------------- | --------------------------------- |
| `Transaction` | Inbound HTTP/RPC request KPIs (throughput, latency, errors, Apdex). Rollup-accelerated.     | Golden signals                    |
| `AppRequest`  | The same KPIs, **including message consumers** (Kafka and similar)                          | Event-driven services             |
| `Span`        | Anything below the request: outbound HTTP, DB, messaging, AEM Sling/JCR/JSP, internal spans | Dependency and code-path analysis |

Count each span kind to see what's instrumented:

```sql
SELECT count(*) AS spans, percentile(duration, 95) AS p95 FROM Span FACET kind
```

Outbound calls by external domain:

```sql
SELECT count(*) AS calls, percentage(*, WHERE status = 'Error') AS errorPct, percentile(duration, 95) AS p95 FROM Span WHERE kind = 'Client' AND externalDomain != '' FACET externalDomain LIMIT 20
```

### Work around nested aggregation

NRQL lets you aggregate an aggregate with a subquery, for example "the average of per-host maximums". SQL doesn't. Here are the alternatives:

| Goal                                    | SQL approach                                                         |
| --------------------------------------- | -------------------------------------------------------------------- |
| A per-group value, then rank the groups | `FACET group` + `ORDER BY alias` + `LIMIT`                           |
| A ratio of two aggregates               | [Formula widget](tutorial-3.md#combine-queries-with-formula-widgets) |
| A value per host, then the "worst host" | `FACET host ORDER BY alias DESC LIMIT 1`                             |
| A per-host average over time            | `FACET host TIMESERIES AUTO`, and read the series                    |

Example: the worst host by p95:

```sql
SELECT percentile(duration, 95) AS p95 FROM Transaction FACET host ORDER BY p95 DESC LIMIT 1
```
