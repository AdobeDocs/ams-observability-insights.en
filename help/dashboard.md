# Dashboard & Query Language

# Examples

---

# Operate

## 12. Performance {#performance}

Every query runs with a 15-second timeout and a bytes-read budget. This doc explains how Observability Insights accelerates queries automatically, and how to design dashboards that stay fast at any time range.

### 12.1 Rollup acceleration (automatic)

`Transaction` charts read pre-aggregated tables when **all** of these hold:

| Condition         | Requirement                                                                                                              |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------ |
| Shape             | Query has `TIMESERIES` (Number/Table totals always read raw for exactness)                                               |
| Window            | ≥ 3 h (1-min tier) · ≥ 1 d (5-min tier) · ≥ 7 d (1-h tier)                                                               |
| Aggregates        | only `count(*)`, `percentage(*, WHERE status = 'Error')`, `apdex(duration, 500)`, `percentile(duration, 50 \| 95 \| 99)` |
| `FACET` / `WHERE` | only on `service`, `appInstance`, `kind`                                                                                 |

`CpuUtilization` and `MemoryUtilization` (`average(value)`) and `JvmCpuRecentUtilization` (`average(value)`) charts with `FACET host` / `WHERE host …` over ≥ 3 h read 1-minute rollups.

**Design tip:** for 7–31-day trend widgets, stick to the rollup-friendly set (p95, error %, Apdex @ 500, count, FACET service) — they stay fast at any range. Save `FACET host`/`name`, `average(duration)` or `urlPath` filters for shorter windows or tables.

### 12.2 General tips

- Use `LIMIT` on every faceted query — especially `FACET urlPath`, `traceId`, `jcr.path` (high cardinality).
- Filter early with `WHERE` on low-cardinality attributes (`service`, `kind`, `status`).
- Prefer `Transaction` over `Span` when you only need inbound requests — far fewer rows.
- `uniqueCount` is exact; on high-cardinality attributes over long windows it is the most expensive aggregate.
- Avoid `TIMESERIES MAX` except for windows of a few minutes.
- Dashboard widgets cache results for 30 s; the refresh button forces a re-run.
- A widget that hits the 15-second timeout or the bytes-read budget errors out — narrow the window, add filters, or reduce facets.

[↑ Back to top](#top)

---

## 13. Troubleshooting {#troubleshooting}

Error messages appear under the editor when you **Validate** or **Run** a query, and on the widget itself on a dashboard. Find the message in the first column.

| Message / symptom                                                            | Cause                                                              | Fix                                                                                                                                    |
| ---------------------------------------------------------------------------- | ------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------- |
| `Unknown entity 'X'`                                                         | Typo or non-catalog name                                           | Check [Data dictionary](#data-dictionary) / the Query guide panel.                                                                     |
| `Attribute 'x' is not available on Entity`                                   | Attribute not on that entity (e.g. `duration` on `CpuUtilization`) | Trace attributes vs metric attributes — see [Data dictionary](#data-dictionary).                                                       |
| `Raw attributes cannot be mixed with aggregates; use FACET to group results` | `SELECT service, count(*)`                                         | `SELECT count(*) … FACET service`.                                                                                                     |
| `FACET and TIMESERIES require an aggregate query`                            | Event query with FACET/TIMESERIES                                  | Use an aggregate.                                                                                                                      |
| `FACET supports at most five attributes`                                     | > 5 facets                                                         | Reduce.                                                                                                                                |
| `average requires a numeric attribute`                                       | e.g. `average(service)`                                            | Use `uniqueCount`/`latest` or a numeric attribute.                                                                                     |
| `rate requires TIMESERIES` / `rate is only available for counter measures`   | `rate()` without TIMESERIES or on a gauge                          | Add `TIMESERIES AUTO`; use a ✔ counter entity ([Curated metric entities](#53-curated-metric-entities)); for gauges use `derivative()`. |
| `rate cannot be combined with other aggregates`                              | Multiple items with `rate()`                                       | One `rate()` per widget; combine with a Formula.                                                                                       |
| `percentOfTotal cannot be combined with TIMESERIES`                          | —                                                                  | Remove TIMESERIES.                                                                                                                     |
| `apdex requires an entity with a status attribute`                           | `apdex` on a metric entity                                         | Use `Transaction`/`Span`/`AppRequest`.                                                                                                 |
| `ORDER BY 'x' must reference a selected value or alias`                      | Ordering by something not selected                                 | Alias the aggregate and order by the alias.                                                                                            |
| `IN requires between 1 and 100 values`                                       | —                                                                  | Split or use `LIKE`.                                                                                                                   |
| `Use IS NULL or IS NOT NULL for null comparisons`                            | `attr = NULL`                                                      | Use `IS NULL`.                                                                                                                         |
| `Unexpected character '{'`                                                   | `{{variable}}` in the Add widget preview or on Save                | See [Tutorial 3, step 8.7](#87-make-dashboards-reusable-with-variables) limitation.                                                    |
| `Unexpected character ';'` / `'"'`                                           | Semicolons or double quotes                                        | Remove `;`; use single quotes.                                                                                                         |
| `Expected a positive duration such as 30 minutes`                            | `TIMESERIES 5m`                                                    | Write `TIMESERIES 5 minutes`.                                                                                                          |
| `Time range cannot exceed 31 days`                                           | Picker > 31 d                                                      | Narrow the window.                                                                                                                     |
| `Select a customer before running a SQL query`                               | No tenant / demo tenant                                            | Pick a customer in the tenant switcher.                                                                                                |
| `All queries in a formula must be the same kind…`                            | Mix of total and TIMESERIES, or a FACET                            | Make all sub-queries plain or all `TIMESERIES AUTO`; remove FACET.                                                                     |
| `Query 'x' must select exactly one metric…`                                  | Sub-query has two aggregates                                       | One aggregate per sub-query.                                                                                                           |
| `Unknown query name 'x' - must be one of: …`                                 | Formula references a missing name                                  | Match names exactly (case-sensitive).                                                                                                  |
| **Save failed**                                                              | A widget query is invalid server-side (often `{{var}}`)            | Open each recently edited widget and **Run** it; fix or remove the invalid one.                                                        |
| _Select a value for {{x}}_                                                   | Variable has no selection                                          | Pick a value in the variable bar.                                                                                                      |
| _No activity in the last …_                                                  | No rows in window                                                  | Widen the window; verify filters with a discovery query ([Data discovery](#101-data-discovery)).                                       |
| Lines shattered into dots                                                    | Bucket smaller than data cadence                                   | Use `TIMESERIES AUTO` (≥ 2 min), or a Bar chart.                                                                                       |
| Facet shows an empty-string group                                            | Attribute missing on some rows                                     | Add `WHERE attr != ''`.                                                                                                                |
| _This legacy SQL panel is no longer run_                                     | Panel from the old raw-SQL builder                                 | Edit → rebuild in SQL.                                                                                                                 |

[↑ Back to top](#top)

---

## 14. Appendix {#appendix}

**In this doc:** [14.1 Limits](#141-limits) · [14.2 Export file format](#142-export-file-format) · [14.3 One-page cheat sheet](#143-one-page-cheat-sheet)

### 14.1 Limits

| Item                             | Limit                                                                   |
| -------------------------------- | ----------------------------------------------------------------------- |
| Query text                       | 4 KB                                                                    |
| `FACET` attributes               | 5                                                                       |
| `LIMIT`                          | 1 – 5 000 (defaults: Number 1, facet 10, events 100, time series 5 000) |
| `IN (…)` values                  | 1 – 100                                                                 |
| Time window                      | 31 days                                                                 |
| Query timeout                    | 15 s (plus a bytes-read budget)                                         |
| `TIMESERIES AUTO` minimum bucket | 2 minutes                                                               |
| Formula                          | 2 – 6 queries; expression ≤ 500 chars; names ≤ 40 chars                 |
| Dashboard                        | ≤ 40 tabs · ≤ 200 widgets per tab · ≤ 50 variables                      |
| Widget                           | title ≤ 200 · subtitle ≤ 500 · unit ≤ 40 · ≤ 10 thresholds              |
| Text widget                      | ≤ 4 000 chars                                                           |
| Variable                         | name ≤ 64 chars · ≤ 500 options/values · query ≤ 4 KB                   |

### 14.2 Export file format

Use **Export** to produce this; **Import** to load it. Useful for version-controlling dashboards or promoting them between tenants.

```json
{
  "name": "Golden Signals",
  "templateVariables": [
    {
      "id": "var-1",
      "name": "svc",
      "label": "Service",
      "kind": "query",
      "multi": true,
      "query": "SELECT count(*) FROM Transaction FACET service LIMIT 200",
      "value": null
    }
  ],
  "pages": [
    {
      "id": "page-1",
      "name": "Overview",
      "widgets": [
        {
          "id": "sec-kpis",
          "title": "SLO KPIs",
          "queryLanguage": "text",
          "text": "",
          "isSection": true,
          "collapsed": false,
          "layout": { "x": 0, "y": 0, "w": 12, "h": 1 }
        },
        {
          "id": "w-error-rate",
          "title": "Error %",
          "queryLanguage": "SQL",
          "query": "SELECT percentage(*, WHERE status = 'Error') AS errorRate FROM Transaction",
          "vizType": "stat",
          "unit": "%",
          "thresholds": [
            { "value": 0, "status": "ok" },
            { "value": 1, "status": "warning" },
            { "value": 5, "status": "error" }
          ],
          "layout": { "x": 0, "y": 1, "w": 3, "h": 2 }
        },
        {
          "id": "w-throughput",
          "title": "Throughput by service",
          "queryLanguage": "SQL",
          "query": "SELECT count(*) AS requests FROM Transaction FACET service TIMESERIES AUTO LIMIT 8",
          "vizType": "area",
          "layout": { "x": 3, "y": 1, "w": 9, "h": 4 }
        },
        {
          "id": "w-availability",
          "title": "Availability %",
          "queryLanguage": "SQL-formula",
          "formulaQueries": [
            {
              "name": "ok",
              "query": "SELECT count(*) AS v FROM Transaction WHERE status != 'Error'"
            },
            {
              "name": "total",
              "query": "SELECT count(*) AS v FROM Transaction"
            }
          ],
          "formulaExpression": "(ok / total) * 100",
          "vizType": "stat",
          "unit": "%",
          "layout": { "x": 0, "y": 3, "w": 3, "h": 2 }
        }
      ]
    }
  ],
  "exportedAt": "2026-10-01T00:00:00.000Z"
}
```

`vizType` values: `timeseries` (Line), `area`, `bar`, `pie`, `stat` (Number), `table`. Threshold `status`: `ok` (green), `warning` (amber), `error` (red).

### 14.3 One-page cheat sheet

```text
SHAPE        SELECT agg FROM Entity                         -> Number
             ... FACET a[, b..e] LIMIT n                     -> Bar / Pie / Table
             ... TIMESERIES AUTO                             -> Line / Area
             ... FACET a TIMESERIES AUTO LIMIT n             -> top-n series
             SELECT attr1, attr2 FROM Entity LIMIT n         -> event Table

AGGREGATES   count(*)  average(x)  sum(x)  min(x)  max(x)  uniqueCount(x)
             percentile(x, 95)  stddev(x)  latest(x[, rank])  earliest(x[, rank])
             percentage(*, WHERE c)  apdex(duration, 500)  percentOfTotal(x)
             rate(value)        -- counters, TIMESERIES only
             derivative(value)  -- gauges,   TIMESERIES only
             agg FILTER (WHERE c) AS alias

WHERE        = != < <= > >=   LIKE / NOT LIKE ('%', '_')   IN / NOT IN (...)
             IS [NOT] NULL   AND OR NOT ( )   strings in 'single quotes'
             string attr present?  ->  attr != ''

ORDER/LIMIT  ORDER BY alias [ASC|DESC]   LIMIT 1..5000

VARIABLES    = {{name}}   IN ({{name}})   numeric: > {{name}}

FORMULA      2-6 named queries, one metric each, no FACET, same shape
             expression: + - * / ( ) numbers names
```

[↑ Back to top](#top)
