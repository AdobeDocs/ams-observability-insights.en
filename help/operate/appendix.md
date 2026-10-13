---
title: Appendix
description: Reference supplementary information for Observability Insights SQL queries and dashboards.
---

## Appendix {#appendix}

**In this doc:** [14.1 Limits](#141-limits) · [14.2 Export file format](#142-export-file-format) · [14.3 One-page cheat sheet](#143-one-page-cheat-sheet)

### Limits

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

### Export file format

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

### One-page cheat sheet

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
