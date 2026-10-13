---
title: Performance
description: Improve query, chart, and dashboard performance in Observability Insights.
---

## Performance {#performance}

Every query runs with a 15-second timeout and a bytes-read budget. This doc explains how Observability Insights accelerates queries automatically, and how to design dashboards that stay fast at any time range.

### Rollup acceleration (automatic)

`Transaction` charts read pre-aggregated tables when **all** of these hold:

| Condition         | Requirement                                                                                                              |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------ |
| Shape             | Query has `TIMESERIES` (Number/Table totals always read raw for exactness)                                               |
| Window            | ≥ 3 h (1-min tier) · ≥ 1 d (5-min tier) · ≥ 7 d (1-h tier)                                                               |
| Aggregates        | only `count(*)`, `percentage(*, WHERE status = 'Error')`, `apdex(duration, 500)`, `percentile(duration, 50 \| 95 \| 99)` |
| `FACET` / `WHERE` | only on `service`, `appInstance`, `kind`                                                                                 |

`CpuUtilization` and `MemoryUtilization` (`average(value)`) and `JvmCpuRecentUtilization` (`average(value)`) charts with `FACET host` / `WHERE host …` over ≥ 3 h read 1-minute rollups.

**Design tip:** for 7–31-day trend widgets, stick to the rollup-friendly set (p95, error %, Apdex @ 500, count, FACET service) — they stay fast at any range. Save `FACET host`/`name`, `average(duration)` or `urlPath` filters for shorter windows or tables.

### General tips

- Use `LIMIT` on every faceted query — especially `FACET urlPath`, `traceId`, `jcr.path` (high cardinality).
- Filter early with `WHERE` on low-cardinality attributes (`service`, `kind`, `status`).
- Prefer `Transaction` over `Span` when you only need inbound requests — far fewer rows.
- `uniqueCount` is exact; on high-cardinality attributes over long windows it is the most expensive aggregate.
- Avoid `TIMESERIES MAX` except for windows of a few minutes.
- Dashboard widgets cache results for 30 s; the refresh button forces a re-run.
- A widget that hits the 15-second timeout or the bytes-read budget errors out — narrow the window, add filters, or reduce facets.
