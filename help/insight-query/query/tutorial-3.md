## Tutorial 3: Advance your dashboarding {#tutorial-3-advance-your-dashboarding}

In this tutorial you'll build the components of a production-grade dashboard: user-experience scores, counter rates, case grouping, histograms, KPI tiles with thresholds, sections, formula widgets and variables.

**In this tutorial:** [8.1 apdex](#81-score-user-experience-with-apdex) · [8.2 rate](#82-turn-counters-into-rates-with-rate) · [8.3 Cases](#83-group-values-into-cases-with-filter) · [8.4 Histograms](#84-build-a-latency-histogram) · [8.5 KPI tiles and sections](#85-build-kpi-tiles-and-sections) · [8.6 Formulas](#86-combine-queries-with-formula-widgets) · [8.7 Variables](#87-make-dashboards-reusable-with-variables) · [8.8 Put it together](#88-put-it-together)

### Score user experience with `apdex`

Apdex turns latency and errors into one score from 0 to 1. With a 500 ms threshold, a request is _satisfied_ at or below 500 ms, _tolerating_ up to 2,000 ms, and _frustrated_ above that **or if it failed**:

```sql
SELECT apdex(duration, 500) AS apdex FROM Transaction
```

Trend it per service:

```sql
SELECT apdex(duration, 500) AS apdex FROM Transaction FACET service TIMESERIES AUTO LIMIT 8
```

Put it in a **Number** widget with thresholds `0 → Red`, `0.7 → Amber`, `0.85 → Green`. The highest matching band wins, so low scores stay red.

### 8.2 Turn counters into rates with `rate`

Counter entities (✔ in the [Data dictionary](#53-curated-metric-entities)) store ever-increasing totals. `rate(value)` converts them to per-second values. It needs `TIMESERIES`, and it must be the only aggregate:

```sql
SELECT rate(value) AS rxBytesPerSec FROM NetworkReceiveRate FACET host TIMESERIES AUTO
```

```sql
SELECT rate(value) AS writeBytesPerSec FROM DiskWriteRate FACET host, device TIMESERIES AUTO LIMIT 10
```

> [!WARNING]
> Always facet `rate()` by `host` (and by `device` for disks), or filter to one host. Without that, the readings from different hosts get mixed together and the result is meaningless.

To get a **request** rate from traces, count requests in fixed buckets instead. This gives requests per minute:

```sql
SELECT count(*) AS requestsPerMinute FROM Transaction TIMESERIES 1 minute
```

### Group values into cases with `FILTER`

NRQL's `FACET CASES` groups rows into named buckets. In SQL, write one `FILTER` column per case. Here are status classes per service:

```sql
SELECT count(*) FILTER (WHERE http.statusCode < 400) AS ok, count(*) FILTER (WHERE http.statusCode >= 400 AND http.statusCode < 500) AS clientErrors, count(*) FILTER (WHERE http.statusCode >= 500) AS serverErrors FROM Transaction FACET service LIMIT 20
```

The same idea works over time, giving one line per case:

```sql
SELECT count(*) FILTER (WHERE urlPath LIKE '/content/%') AS pages, count(*) FILTER (WHERE urlPath LIKE '/bin/%') AS servlets, count(*) FILTER (WHERE urlPath LIKE '/graphql/%') AS graphql FROM Transaction TIMESERIES AUTO
```

### Build a latency histogram

SQL has no `histogram()` function. Build the buckets yourself with `FILTER`, and render the result as a **Bar** or **Table** widget:

```sql
SELECT count(*) FILTER (WHERE duration <= 100) AS 'le 100ms', count(*) FILTER (WHERE duration > 100 AND duration <= 250) AS '100-250ms', count(*) FILTER (WHERE duration > 250 AND duration <= 500) AS '250-500ms', count(*) FILTER (WHERE duration > 500 AND duration <= 1000) AS '0.5-1s', count(*) FILTER (WHERE duration > 1000 AND duration <= 2000) AS '1-2s', count(*) FILTER (WHERE duration > 2000) AS 'over 2s' FROM Transaction
```

Add `FACET service` to compare the distributions of several services side by side.

### Build KPI tiles and sections

A good dashboard starts with a row of KPI tiles, then groups detail charts in sections.

1. Add four **Number** widgets, each 3 columns wide:

   | Title    | Query                                                                       | Unit | Thresholds                               |
   | -------- | --------------------------------------------------------------------------- | ---- | ---------------------------------------- |
   | Requests | `SELECT count(*) AS requests FROM Transaction`                              |      |                                          |
   | Error %  | `SELECT percentage(*, WHERE status = 'Error') AS errorPct FROM Transaction` | `%`  | `0 → Green`, `1 → Amber`, `5 → Red`      |
   | p95      | `SELECT percentile(duration, 95) AS p95 FROM Transaction`                   | `ms` | `0 → Green`, `800 → Amber`, `2000 → Red` |
   | Apdex    | `SELECT apdex(duration, 500) AS apdex FROM Transaction`                     |      | `0 → Red`, `0.7 → Amber`, `0.85 → Green` |

2. Add a **Text** widget, tick _Make this a section header_, and call it **LATENCY**. Add latency charts below it.
3. Add another section header, **ERRORS**, followed by error charts. Collapse a section with its chevron to focus on the other.
4. Use **tabs** for different audiences, for example _Overview_, _AEM_, _Infrastructure_. Only the visible tab runs its queries.

### Combine queries with formula widgets

SQL can't do math in `SELECT` and has no subqueries. **Formula widgets** fill that gap: they run 2–6 named queries and combine their results with `+ - * / ( )`.

**Example: requests per host over time** (Formula, Line)

1. **Add widget → Formula**.
2. Name the first query `requests` and the second `hosts`.
3. Enter the queries and the formula, then click **Run formula**.

Query `requests`:

```sql
SELECT count(*) AS v FROM Transaction TIMESERIES AUTO
```

Query `hosts`:

```sql
SELECT uniqueCount(host) AS v FROM Transaction TIMESERIES AUTO
```

Formula:

```text
requests / hosts
```

**Example: share of time spent in the database** (Formula, Number, unit `%`)

Query `db`:

```sql
SELECT sum(duration) AS v FROM Span WHERE db.system != ''
```

Query `server`:

```sql
SELECT sum(duration) AS v FROM Transaction
```

Formula:

```text
(db / server) * 100
```

Rules for formula queries:

| Rule            | Details                                                                                                              |
| --------------- | -------------------------------------------------------------------------------------------------------------------- |
| Names           | Letters, digits and `_`, starting with a letter or `_`; up to 40 characters; unique                                  |
| Expression      | `+ - * / ( )`, numbers and query names; up to 500 characters                                                         |
| Same shape      | All queries are plain totals (Number), **or** all are `TIMESERIES` with the **same bucket size** (Line / Area / Bar) |
| One metric each | Each query returns exactly one aggregate and no `FACET`                                                              |
| Missing data    | A missing value or a division by zero produces an empty point, never `Infinity` or `NaN`                             |

> [!TIP]
> Use `TIMESERIES AUTO` in every query of a time-series formula. Because the queries share one window, they always resolve to the same bucket size.

### Make dashboards reusable with variables

Variables let one dashboard serve many services, tiers or thresholds. Each variable is a picker in the variable bar. Reference it as `{{name}}` in widget titles and text widgets (and, once the limitation below is fixed, in widget SQL).

**Create a variable** (_org and super admins_): click **+ Variable**.

| Field                               | Notes                                                                                                                                                   |
| ----------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Name**                            | Letters, digits and `_`, starting with a letter or `_`. Must be unique. Used as `{{name}}`.                                                             |
| **Label**                           | What the picker shows (optional).                                                                                                                       |
| **Type**                            | **Custom list** (one value per line), **From a query** (a SQL query with one `FACET`; its distinct values become options), or **Text box** (free text). |
| **Allow selecting multiple values** | Expands to a comma-separated list. Use it with `IN (…)`.                                                                                                |
| **Values are numbers**              | Injects values unquoted (`500` instead of `'500'`).                                                                                                     |

**How substitution works**

| Selection                          | `{{svc}}` becomes                                       |
| ---------------------------------- | ------------------------------------------------------- |
| Single string `checkout`           | `'checkout'`                                            |
| Multiple values `checkout`, `cart` | `'checkout', 'cart'`                                    |
| Number `500`                       | `500`                                                   |
| Value containing `'`               | Escaped as `''`                                         |
| Nothing selected                   | The widget waits and shows _Select a value for {{svc}}_ |

```text
WHERE service = {{svc}}            single select
WHERE service IN ({{svc}})         multiple select
WHERE duration > {{slowMs}}        numeric text box
```

For a query-driven variable, **always add a `LIMIT`**. A faceted query returns only the top 10 values by default. This lists up to 200 services:

```sql
SELECT count(*) FROM Transaction FACET service LIMIT 200
```

Options refresh when the tenant or time picker changes, and they're sorted alphabetically. When the current selection is no longer valid, the first value (single select) or all values (multiple select) are selected automatically. A variable's query can't reference other variables.

> [!WARNING]
> **Current build limitation.** Substitution happens in the dashboard grid, but the **Add widget** preview and the **server-side save check** receive the raw `{{name}}` token and reject it with _Unexpected character '{'_. In practice:
>
> - `{{variables}}` in **widget titles** and **text widgets** work end to end.
> - `{{variables}}` inside **widget SQL** render correctly after an **Import**, but the dashboard can't be **saved** while such widgets exist.
>
> Until this is fixed, build per-context widgets with literal filters (for example `WHERE service = 'checkout'`) and use **Duplicate** to clone them.

### Put it together

You now have every building block used in the [Dashboard blueprints](#dashboard-blueprints): KPI tiles with thresholds, sections, percentiles, Apdex, rates, case columns, histograms, formulas and variables. The blueprints show how to combine them into complete golden-signal, error-triage, AEM, infrastructure and JVM dashboards.
