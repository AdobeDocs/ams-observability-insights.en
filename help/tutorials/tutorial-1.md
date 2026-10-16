---
title: "Tutorial 1: Your first queries"
description: Write your first SQL queries against Observability Insights telemetry.
---

## Tutorial 1: Your first queries {#tutorial-1-your-first-queries}

Welcome to SQL. In this tutorial you'll write your first queries, control the time window, plot trends, filter and group results, and put your first chart on a dashboard. Each step builds on the previous one.

### Before you begin

- You need access to Observability Insights, and at least one customer tenant sending traces or metrics.
- Select that **customer** in the tenant switcher. SQL doesn't run while **All accounts** is selected.
- Open **Explore** and switch to the **SQL** tab. Use **Run** (⌘/Ctrl + Enter) after each query.

### Query events with `SELECT`, `FROM` and `LIMIT`

Every query needs two clauses: `SELECT` says what to return, and `FROM` says which entity to read. Start with inbound requests:

```sql
SELECT timestamp, service, name, duration FROM Transaction
```

This is an **event query**. Each row is one request. Without a `LIMIT` you get up to 100 rows. Ask for fewer:

```sql
SELECT timestamp, service, name, duration FROM Transaction LIMIT 10
```

Rows are sorted by the first column, descending, so putting `timestamp` first gives you the newest requests at the top.

>[!NOTE]
>
> SQL has no `SELECT *`. Listing attributes keeps results small and readable. The [Data dictionary](#data-dictionary), and the autocomplete in the editor, show what's available.

### Aggregate your data

Most dashboards don't show individual events. They summarize them. Replace the attributes with an **aggregator function**:

```sql
SELECT count(*) FROM Transaction
```

That's the number of inbound requests in the window. Try a few more:

```sql
SELECT average(duration) FROM Transaction
```

```sql
SELECT max(duration) FROM Transaction
```

```sql
SELECT uniqueCount(host) FROM Transaction
```

### Return several aggregates at once

Separate aggregates with commas, and name each one with `AS`:

```sql
SELECT count(*) AS requests, average(duration) AS avgMs, max(duration) AS maxMs FROM Transaction
```

>[!TIP]
>
> Aliases become column headers and legend labels, and they're what you reference in `ORDER BY`. Get into the habit of naming every aggregate.

### Set the time window

By default a query covers the **last hour**. Use `SINCE` to look further back:

```sql
SELECT count(*) AS requests FROM Transaction SINCE 1 day ago
```

Use `UNTIL` to end the window in the past. This counts yesterday's traffic, from 48 to 24 hours ago:

```sql
SELECT count(*) AS requests FROM Transaction SINCE 2 days ago UNTIL 1 day ago
```

Units can be `second`, `minute`, `hour`, `day` or `week`. The window can be at most 31 days.

> **Important**
>
> In **Explore** and on **dashboards**, the **time picker** at the top of the page controls the window and overrides `SINCE`/`UNTIL`. To look at another period, change the picker. The rest of this guide leaves `SINCE` out of dashboard queries for that reason.

### Plot trends with `TIMESERIES`

Add `TIMESERIES` to turn a single number into a line over time:

```sql
SELECT count(*) AS requests FROM Transaction TIMESERIES AUTO
```

`AUTO` chooses a bucket size so the chart has about 120 points (2 minutes for a 1-hour window, 15 minutes for 24 hours). You can also set the bucket explicitly:

```sql
SELECT average(duration) AS avgMs FROM Transaction TIMESERIES 5 minutes
```

### Filter with `WHERE`

`WHERE` keeps only the rows you care about. Count only failed requests:

```sql
SELECT count(*) AS errors FROM Transaction WHERE status = 'Error'
```

Combine conditions with `AND`, `OR` and parentheses:

```sql
SELECT count(*) AS slowOrFailed FROM Transaction WHERE service = 'checkout' AND (status = 'Error' OR duration > 2000)
```

Use `LIKE` with `%` for partial matches:

```sql
SELECT count(*) AS contentRequests FROM Transaction WHERE urlPath LIKE '/content/%'
```

>[!NOTE]
>
> String values are case sensitive and use single quotes: `'Error'`, not `"Error"` or `'error'`. Replace `checkout` with a service from your tenant. The next step shows how to list them.

### Group with `FACET`

`FACET` groups results, like SQL's `GROUP BY`. List your services and how busy each one is:

```sql
SELECT count(*) AS requests FROM Transaction FACET service
```

A faceted query returns the top 10 groups by default. Use `LIMIT` to see more:

```sql
SELECT count(*) AS requests, average(duration) AS avgMs FROM Transaction FACET service LIMIT 50
```

Combine `FACET` and `TIMESERIES` to get one line per group:

```sql
SELECT count(*) AS requests FROM Transaction FACET service TIMESERIES AUTO LIMIT 5
```

Here, `LIMIT 5` keeps the five busiest series.

### Save your query to a dashboard

1. Open **Dashboards → Create dashboard → Build a custom dashboard** and name it _My first dashboard_.
2. Click **Add widget**, choose **Single query**, and switch the toggle to **SQL**.
3. Paste the query from step 6.7 and click **Run query**.
4. Set the **Title** to _Requests by service_ and choose the **Line** chart type.
5. Click **Save widget**, then **Save** in the top bar.

You've built your first dashboard.
