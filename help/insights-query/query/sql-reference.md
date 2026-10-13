## SQL reference {#sql-reference}

This reference covers SQL syntax, clauses, functions and operators. Expand each entry for details and examples.

**In this doc:** [Query components](#41-query-components) · [Required clauses](#42-required-clauses) · [Optional clauses](#43-optional-clauses) · [Aggregator functions](#44-aggregator-functions) · [Operators and conditions](#45-operators-and-conditions) · [Type handling](#46-type-handling-blank-vs-null) · [SQL to SQL](#47-translate-sql-to-sql) · [Importing existing dashboards](#48-importing-existing-dashboards-to-insights)

### Query components

```text
SELECT <select-item> [, <select-item> ...]
FROM   <Entity>
[WHERE      <condition>]
[FACET      <attribute> [, <attribute> ...]]          -- up to 5
[SINCE      <n> <unit> [AGO]]
[UNTIL      NOW | <n> <unit> [AGO]]
[TIMESERIES AUTO | MAX | <n> <unit>]
[ORDER BY   <alias-or-selected-name> [ASC | DESC]]
[LIMIT      <1..5000>]

<select-item> := <function>(<args>) [FILTER (WHERE <condition>)] [AS <alias>]
               | <attribute> [AS <alias>]
<unit>        := second | seconds | minute | minutes | hour | hours | day | days | week | weeks
```

> [!NOTE]
> The `--` annotations above are documentation only. SQL itself doesn't accept comments.

### Required clauses

<details>
<summary><code>SELECT</code> — what to return</summary>

`SELECT` lists either **raw attributes** (an event query) or **aggregator functions** (an aggregate query). You can't mix the two.

| Form         | Example                                            | Result                                            |
| ------------ | -------------------------------------------------- | ------------------------------------------------- |
| Attributes   | `SELECT timestamp, name, duration`                 | Rows of individual events (default 100)           |
| Aggregates   | `SELECT count(*), average(duration)`               | One value per aggregate (per facet / time bucket) |
| With aliases | `SELECT count(*) AS requests`                      | Column label `requests`                           |
| Quoted alias | `SELECT percentile(duration, 95) AS 'p95 latency'` | Label with spaces                                 |

Event list:

```sql
SELECT timestamp, service, name, duration, status, traceId FROM Transaction WHERE status = 'Error' LIMIT 50
```

Several aggregates:

```sql
SELECT count(*) AS requests, average(duration) AS avgMs, max(duration) AS maxMs FROM Transaction
```

Rules:

- `SELECT *` isn't supported. List the attributes you want.
- Math between attributes or aggregates (`sum(a) / count(*)`) isn't supported. Use a [Formula widget](#86-combine-queries-with-formula-widgets).
- Mixing attributes and aggregates fails with _Raw attributes cannot be mixed with aggregates; use FACET to group results_.

</details>

<details>
<summary><code>FROM</code> — which entity to query</summary>

`FROM` names exactly **one** entity from the [Data dictionary](#data-dictionary). The entity name isn't case sensitive.

```sql
SELECT count(*) FROM Transaction
```

```sql
FROM Span SELECT count(*) FACET kind
```

Each entity carries a fixed scope. For example, `Transaction` is always server spans, and `CpuUtilization` is always the CPU-busy metric with `value` converted to percent. You can't query more than one entity in a single query. To combine entities, use a [Formula widget](#86-combine-queries-with-formula-widgets).

</details>

### Optional clauses

<details>
<summary><code>AS</code> — label a column</summary>

Use `AS` to give an aggregate or attribute a readable name. The alias becomes the column header, legend label and the name you reference in `ORDER BY`.

```sql
SELECT count(*) AS requests, percentile(duration, 95) AS p95 FROM Transaction FACET service ORDER BY p95 DESC LIMIT 10
```

Use a single-quoted string for labels with spaces:

```sql
SELECT percentile(duration, 95) AS 'p95 latency (ms)' FROM Transaction TIMESERIES AUTO
```

</details>

<details>
<summary><code>FACET</code> — group results</summary>

`FACET` groups results by up to **5** attributes, like SQL `GROUP BY`.

```sql
SELECT count(*) FROM Transaction FACET service, http.method, http.statusCode LIMIT 30
```

| Query                        | What `LIMIT n` means                                                                                                          | Default                           |
| ---------------------------- | ----------------------------------------------------------------------------------------------------------------------------- | --------------------------------- |
| `FACET` without `TIMESERIES` | Number of rows                                                                                                                | 10                                |
| `FACET` with `TIMESERIES`    | **Top n series**, ranked by the sum of the first aggregate over the whole window. Every time bucket is kept for those series. | All series (capped at 5,000 rows) |

Faceted time series, top 5:

```sql
SELECT count(*) AS requests FROM Transaction FACET service TIMESERIES AUTO LIMIT 5
```

Facet values can be strings or numbers. Blank strings form their own group. Exclude them with `WHERE attr != ''`.

</details>

<details>
<summary><code>FILTER</code> — conditional aggregates</summary>

Append `FILTER (WHERE condition)` to any aggregate to compute it over a subset of rows. This lets you compare several conditions in one pass.

```sql
SELECT count(*) AS total, count(*) FILTER (WHERE status = 'Error') AS errors, percentile(duration, 95) FILTER (WHERE http.method = 'GET') AS getP95 FROM Transaction FACET service LIMIT 20
```

`FILTER` isn't allowed with `apdex`, `rate` or `derivative`. `percentage` already takes its own condition.

</details>

<details>
<summary><code>LIMIT</code> — how many rows or series</summary>

`LIMIT` takes a positive integer. Values above 5,000 are clamped to 5,000.

| Result shape | Default    |
| ------------ | ---------- |
| Billboard    | 1          |
| Facet        | 10         |
| Time series  | 5,000 rows |
| Events       | 100        |

```sql
SELECT count(*) FROM Transaction FACET urlPath LIMIT 50
```

> [!TIP]
> Always set a `LIMIT` on query-driven variables and on high-cardinality facets (`urlPath`, `traceId`, `jcr.path`, `url`).

</details>

<details>
<summary><code>ORDER BY</code> — sort results</summary>

`ORDER BY` sorts by a **selected alias or selected attribute**. The default direction is `DESC`.

```sql
SELECT count(*) AS requests, percentile(duration, 95) AS p95 FROM Transaction FACET name ORDER BY p95 DESC LIMIT 20
```

| Result shape      | Default order                                                               |
| ----------------- | --------------------------------------------------------------------------- |
| Billboard / facet | First aggregate, `DESC`                                                     |
| Time series       | Time, `ASC`                                                                 |
| Events            | First selected column, `DESC` (so put `timestamp` first for "newest first") |

Rules:

- Referencing something that isn't selected fails with _ORDER BY 'x' must reference a selected value or alias_.
- Don't add `ORDER BY` to a `TIMESERIES` query. It overrides the time ordering that charts need.

</details>

<details>
<summary><code>SINCE</code> — start of the window</summary>

`SINCE n unit [AGO]` sets how far back the window begins. Units: `second`, `minute`, `hour`, `day`, `week` (singular or plural). `AGO` is optional.

```sql
SELECT count(*) FROM Transaction SINCE 6 hours ago
```

- The default is `SINCE 1 hour ago`.
- The window can't exceed **31 days** (_Time range cannot exceed 31 days_).
- Absolute dates (`SINCE '2026-01-01'`) and `TODAY` / `YESTERDAY` aren't supported.

> [!IMPORTANT]
> On dashboards and in Explore, the **time picker overrides** `SINCE` and `UNTIL`. See [Time window precedence](#time-window-precedence).

</details>

<details>
<summary><code>TIMESERIES</code> — plot over time</summary>

`TIMESERIES` returns one row per time bucket (per facet value), which is ready for Line, Area or Bar charts.

| Form                   | Bucket size                                                                                                                                   |
| ---------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| `TIMESERIES 5 minutes` | Exactly what you ask for (any unit)                                                                                                           |
| `TIMESERIES AUTO`      | About window ÷ 120, rounded up to 1s / 5s / 10s / 15s / 30s / 1m / 5m / 10m / 15m / 30m / 1h / 3h / 6h / 12h / 1d. **Never below 2 minutes.** |
| `TIMESERIES MAX`       | 1-second buckets. Use only over a few minutes.                                                                                                |

Typical `AUTO` buckets:

| Window | 1 h   | 6 h   | 24 h   | 7 d | 30 d |
| ------ | ----- | ----- | ------ | --- | ---- |
| Bucket | 2 min | 5 min | 15 min | 3 h | 6 h  |

```sql
SELECT percentile(duration, 95) AS p95 FROM Transaction TIMESERIES AUTO
```

```sql
SELECT count(*) FROM Transaction TIMESERIES 1 hour SINCE 1 day ago
```

> [!TIP]
> Prefer `AUTO` on dashboards. It keeps about 120 points whatever the time picker says. It also avoids "shattered" per-host lines, because host metrics are scraped about every 90 seconds.

</details>

<details>
<summary><code>UNTIL</code> — end of the window</summary>

`UNTIL NOW` (default) or `UNTIL n unit [AGO]`. Combine it with `SINCE` to look at a past window:

```sql
SELECT count(*) AS yesterday FROM Transaction SINCE 2 days ago UNTIL 1 day ago
```

`SINCE` must be earlier than `UNTIL`. Otherwise the query fails with _SINCE must be earlier than UNTIL_.

</details>

<details>
<summary><code>WHERE</code> — filter rows</summary>

`WHERE` keeps only rows that match a condition. Combine conditions with `AND`, `OR`, `NOT` and parentheses. See [Operators and conditions](#45-operators-and-conditions).

```sql
SELECT count(*) FROM Transaction WHERE service = 'checkout' AND (status = 'Error' OR http.statusCode >= 500)
```

`WHERE` can appear only once. Put all conditions in it.

</details>

### Aggregator functions

| Function                               | Arguments                           | Returns                | Works on                              |
| -------------------------------------- | ----------------------------------- | ---------------------- | ------------------------------------- |
| [`apdex`](#fn-apdex)                   | numeric duration, threshold `T`     | 0–1                    | Trace entities                        |
| [`average` / `avg`](#fn-average)       | numeric                             | Same unit as the input | Any                                   |
| [`count`](#fn-count)                   | `*` or attribute                    | Number of rows         | Any                                   |
| [`derivative`](#fn-derivative)         | numeric                             | Change per second      | Any (needs `TIMESERIES`)              |
| [`earliest`](#fn-latest)               | any [, rank]                        | Attribute value        | Any                                   |
| [`latest`](#fn-latest)                 | any [, rank]                        | Attribute value        | Any                                   |
| [`max` / `min`](#fn-minmax)            | any                                 | Same unit as the input | Any                                   |
| [`percentage`](#fn-percentage)         | `*` or attribute, `WHERE` condition | %                      | Any                                   |
| [`percentile`](#fn-percentile)         | numeric, level `p`                  | Same unit as the input | Any                                   |
| [`percentOfTotal`](#fn-percentoftotal) | numeric                             | %                      | Any (no `TIMESERIES`)                 |
| [`rate`](#fn-rate)                     | numeric counter                     | Per second             | Counter entities (needs `TIMESERIES`) |
| [`stddev`](#fn-stddev)                 | numeric                             | Same unit as the input | Any                                   |
| [`sum`](#fn-sum)                       | numeric                             | Same unit as the input | Any                                   |
| [`uniqueCount`](#fn-uniquecount)       | any                                 | Distinct values        | Any                                   |

<a id="fn-apdex"></a>

<details>
<summary><code>apdex(attribute, T)</code></summary>

Calculates an [Apdex](https://www.apdex.org/) score from 0 (frustrated) to 1 (satisfied), using threshold `T` in the attribute's unit (milliseconds for `duration`).

| Bucket     | Condition                            | Weight |
| ---------- | ------------------------------------ | ------ |
| Satisfied  | `duration ≤ T` and not an error      | 1      |
| Tolerating | `T < duration ≤ 4T` and not an error | 0.5    |
| Frustrated | `duration > 4T`, **or any error**    | 0      |

```sql
SELECT apdex(duration, 500) AS apdex FROM Transaction FACET service
```

- Works only on trace entities (`Transaction`, `AppRequest`, `Span`).
- `T` must be greater than 0. `FILTER` isn't allowed.
- `apdex(duration, 500)` on `Transaction` is rollup-accelerated over long windows.

</details>

<a id="fn-average"></a>

<details>
<summary><code>average(attribute)</code> / <code>avg(attribute)</code></summary>

The arithmetic mean of a numeric attribute.

```sql
SELECT average(duration) AS avgMs FROM Transaction FACET service TIMESERIES AUTO
```

</details>

<a id="fn-count"></a>

<details>
<summary><code>count(*)</code> / <code>count(attribute)</code></summary>

- `count(*)` counts rows.
- `count(attribute)` counts rows where the attribute is **not NULL**. For string attributes, missing values are blank (`''`) rather than NULL, so they are still counted (see [Type handling](#46-type-handling-blank-vs-null)).

```sql
SELECT count(*) AS requests, count(http.statusCode) AS withStatusCode FROM Transaction
```

</details>

<a id="fn-derivative"></a>

<details>
<summary><code>derivative(attribute)</code></summary>

The per-second change of `average(attribute)` between consecutive buckets: (this bucket − previous bucket) ÷ bucket seconds. The result can be negative. Use it on gauges to see whether a value is growing or shrinking.

```sql
SELECT derivative(value) AS heapGrowthPerSec FROM JvmHeapUsed WHERE host = 'publish-1' TIMESERIES 5 minutes
```

- Requires `TIMESERIES`, and must be the **only** aggregate. `FILTER` isn't allowed.
- The first bucket has no predecessor.

</details>

<a id="fn-latest"></a>

<details>
<summary><code>latest(attribute [, rankAttribute])</code> / <code>earliest(…)</code></summary>

- `latest(attr)` returns the value from the most recent row. `earliest(attr)` returns the value from the oldest row.
- With a second argument, the row is chosen by that attribute instead of time. `latest(traceId, duration)` returns the trace ID of the **slowest** request. `earliest(traceId, duration)` returns the fastest.

```sql
SELECT latest(value) AS current FROM MemoryUtilization FACET host
```

```sql
SELECT max(duration) AS slowestMs, latest(traceId, duration) AS slowestTrace FROM Transaction FACET name ORDER BY slowestMs DESC LIMIT 10
```

</details>

<a id="fn-minmax"></a>

<details>
<summary><code>max(attribute)</code> / <code>min(attribute)</code></summary>

The largest or smallest value. Works on numbers, strings and timestamps.

```sql
SELECT min(duration) AS fastestMs, max(duration) AS slowestMs, max(timestamp) AS lastSeen FROM Transaction FACET service
```

</details>

<a id="fn-percentage"></a>

<details>
<summary><code>percentage(*, WHERE condition)</code></summary>

The share of rows that match `condition`, from 0 to 100.

```sql
SELECT percentage(*, WHERE status = 'Error') AS errorPct FROM Transaction FACET service TIMESERIES AUTO
```

```sql
SELECT percentage(*, WHERE duration > 2000) AS slowPct FROM Transaction
```

`percentage` has its own condition, so it doesn't take `FILTER`.

</details>

<a id="fn-percentile"></a>

<details>
<summary><code>percentile(attribute, p)</code></summary>

The approximate p-th percentile, where `0 < p ≤ 100`. Use one `percentile` per level.

```sql
SELECT percentile(duration, 50) AS p50, percentile(duration, 95) AS p95, percentile(duration, 99) AS p99 FROM Transaction TIMESERIES AUTO
```

p50, p95 and p99 on `Transaction` are rollup-accelerated over long windows.

</details>

<a id="fn-percentoftotal"></a>

<details>
<summary><code>percentOfTotal(attribute)</code></summary>

Each facet row's share of the **total sum** of a numeric attribute. The rows add up to 100%.

```sql
SELECT sum(duration) AS totalMs, percentOfTotal(duration) AS shareOfTime FROM Transaction FACET name LIMIT 15
```

Not allowed with `TIMESERIES`.

</details>

<a id="fn-rate"></a>

<details>
<summary><code>rate(attribute)</code></summary>

The per-second rate of a **cumulative counter**. In each bucket, `rate` takes the highest counter reading per series and diffs it against the previous bucket. Counter resets are floored at 0, and the first bucket is 0.

```sql
SELECT rate(value) AS bytesPerSec FROM NetworkReceiveRate FACET host TIMESERIES AUTO
```

- Works only on counter entities (marked ✔ in the [Data dictionary](#53-curated-metric-entities)).
- Requires `TIMESERIES`, and must be the **only** aggregate. `FILTER` isn't allowed.

> [!WARNING]
> Always `FACET host`, or filter to a single host. An unfaceted `rate()` across many hosts compares different hosts' counters and produces meaningless numbers. Disk counters are per device, so `FACET host, device` is the most precise.

</details>

<a id="fn-stddev"></a>

<details>
<summary><code>stddev(attribute)</code></summary>

The sample standard deviation. Use it with `average` to judge how consistent latency is.

```sql
SELECT average(duration) AS avgMs, stddev(duration) AS stddevMs FROM Transaction FACET name LIMIT 20
```

</details>

<a id="fn-sum"></a>

<details>
<summary><code>sum(attribute)</code></summary>

The total of a numeric attribute.

```sql
SELECT sum(callCount) AS calls FROM SpanBreakdown FACET segment LIMIT 20
```

</details>

<a id="fn-uniquecount"></a>

<details>
<summary><code>uniqueCount(attribute)</code></summary>

The **exact** number of distinct values.

```sql
SELECT uniqueCount(host) AS hosts, uniqueCount(traceId) AS traces FROM Transaction FACET service
```

</details>

Function constraints at a glance:

| Constraint                            | Functions                                                                                 |
| ------------------------------------- | ----------------------------------------------------------------------------------------- |
| Numeric attribute required            | `average`, `sum`, `percentile`, `stddev`, `percentOfTotal`, `rate`, `derivative`, `apdex` |
| No `FILTER (WHERE …)`                 | `apdex`, `rate`, `derivative`, `percentage`                                               |
| Requires `TIMESERIES`, only aggregate | `rate`, `derivative`                                                                      |
| Not with `TIMESERIES`                 | `percentOfTotal`                                                                          |
| Trace entities only                   | `apdex`                                                                                   |
| Counter entities only                 | `rate`                                                                                    |

### Operators and conditions

| Operator                  | Meaning                                                                    | Example                                  |
| ------------------------- | -------------------------------------------------------------------------- | ---------------------------------------- |
| `=`                       | Equals                                                                     | `status = 'Error'`                       |
| `!=`, `<>`                | Not equal                                                                  | `kind != 'Internal'`                     |
| `<`, `<=`, `>`, `>=`      | Comparison                                                                 | `http.statusCode >= 500`                 |
| `LIKE`                    | Wildcard match: `%` = any characters, `_` = one character. Case sensitive. | `urlPath LIKE '/content/%'`              |
| `NOT LIKE`                | Negated wildcard                                                           | `urlPath NOT LIKE '%.css'`               |
| `IN (…)`                  | Matches any of 1–100 values                                                | `service IN ('checkout', 'cart')`        |
| `NOT IN (…)`              | Matches none of the values                                                 | `http.method NOT IN ('OPTIONS', 'HEAD')` |
| `IS NULL` / `IS NOT NULL` | Null check                                                                 | `http.statusCode IS NOT NULL`            |
| `AND`, `OR`, `NOT`        | Logic                                                                      | `NOT (status = 'Ok')`                    |
| `( … )`                   | Grouping                                                                   | `(a = 1 OR b = 2) AND c = 3`             |

```sql
SELECT count(*) FROM Transaction WHERE (status = 'Error' OR duration > 2000) AND NOT host LIKE '%dispatcher%' AND http.method IN ('GET', 'POST')
```

> [!WARNING]
> `attr = NULL` is an error. Use `attr IS NULL`.

### Type handling: blank vs NULL

Every attribute has a type in the catalog: `string`, `number`, `boolean` or `timestamp`. SQL type-checks functions against it (for example, `average(service)` is rejected). No type conversion functions are needed or available.

> [!IMPORTANT]
> **Blank vs NULL.** Most string attributes come from OpenTelemetry attribute maps, where a _missing_ key reads as an empty string `''`, not `NULL`. To keep only spans that carry an attribute, use **`attr != ''`** rather than `IS NOT NULL`. Numeric attributes such as `http.statusCode` are genuinely `NULL` when absent, so `IS NOT NULL` works for them.

```sql
SELECT count(*) AS jcrCalls FROM Span WHERE jcr.operation != '' FACET jcr.operation
```

### Translate SQL to SQL

| SQL                                                          | SQL                                                                 |
| ------------------------------------------------------------ | ------------------------------------------------------------------- |
| `WHERE ts > now() - INTERVAL 1 HOUR`                         | Nothing; the time picker (or `SINCE 1 hour ago`) sets the window    |
| `GROUP BY service`                                           | `FACET service`                                                     |
| `GROUP BY toStartOfInterval(ts, INTERVAL 5 MINUTE), service` | `FACET service TIMESERIES 5 minutes`                                |
| `quantile(0.95)(duration)`                                   | `percentile(duration, 95)`                                          |
| `countIf(status = 'Error')`                                  | `count(*) FILTER (WHERE status = 'Error')`                          |
| `100 * countIf(x) / count()`                                 | `percentage(*, WHERE x)`                                            |
| `count(DISTINCT host)`                                       | `uniqueCount(host)`                                                 |
| `argMax(traceId, duration)`                                  | `latest(traceId, duration)`                                         |
| `x / sum(x) OVER ()`                                         | `percentOfTotal(x)`                                                 |
| `ORDER BY p95 DESC LIMIT 20`                                 | `ORDER BY p95 DESC LIMIT 20`                                        |
| `HAVING count() > 100`                                       | _Not supported_. Sort and `LIMIT` instead.                          |
| `JOIN` / subquery                                            | _Not supported_. Use a **Formula** widget.                          |
| `CASE WHEN`                                                  | _Not supported_. Use `FILTER` columns, or `WHERE … LIKE` + `FACET`. |

### Importing existing dashboards to Insights

If you know the existing query, following is the migration detail

| Existing Query                              | SQL                                                                | Notes                                                        |
| ------------------------------------------- | ------------------------------------------------------------------ | ------------------------------------------------------------ |
| `FROM Transaction SELECT count(*)`          | Same                                                               | `FROM`-first order is supported                              |
| `SINCE 1 day ago`                           | Same                                                               | Ignored on dashboards; the picker wins                       |
| `TIMESERIES AUTO` / `MAX` / `5 minutes`     | Same                                                               | `AUTO` floor is 2 minutes                                    |
| `FACET appName`                             | `FACET appName` / `FACET service`                                  |                                                              |
| `percentage(count(*), WHERE error IS true)` | `percentage(*, WHERE status = 'Error')`                            |                                                              |
| `filter(count(*), WHERE …)`                 | `count(*) FILTER (WHERE …)`                                        | SQL-standard form                                            |
| `apdex(duration, t: 0.5)`                   | `apdex(duration, 500)`                                             | `duration` is in **ms**                                      |
| `latest(x)` / `earliest(x)`                 | Same                                                               | Plus a ranked form: `latest(x, rankAttr)`                    |
| `uniqueCount(x)`                            | Same                                                               | Exact                                                        |
| `rate(count(*), 1 minute)`                  | `count(*)` with `TIMESERIES 1 minute`                              | SQL `rate()` is for counters only                            |
| `derivative(x, 1 minute)`                   | `derivative(x)` + `TIMESERIES`                                     | Per second                                                   |
| `uniques(x)`                                | `count(*) … FACET x LIMIT n`                                       |                                                              |
| `keyset()` / `SHOW EVENT TYPES`             | [Discovery queries](#101-data-discovery), or the Query guide panel |                                                              |
| `histogram(duration, …)`                    | `FILTER` buckets                                                   | See [Tutorial 3](#84-build-a-latency-histogram)              |
| `FACET CASES (…)`                           | `FILTER` columns                                                   | See [Tutorial 3](#83-group-values-into-cases-with-filter)    |
| `COMPARE WITH 1 week ago`                   | Not supported                                                      | Change the time picker, or use `SINCE … UNTIL …` via the API |
| `RLIKE`                                     | `LIKE` / `IN`                                                      |                                                              |
| Math in `SELECT` / nested aggregation       | **Formula** widget                                                 |                                                              |
| `WITH TIMEZONE`, `EXTRAPOLATE`, `SLIDE BY`  | Not supported                                                      |                                                              |
