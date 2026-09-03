# Observability Insights Public API

The Observability Insights Public API lets you pull your own observability data — request overviews, service catalogs, traces, and metrics — directly into your own tools, scripts, and dashboards.

- **API Base URL (API_BASE_URL):** `https://insights.adobecqms.net/`
- **Format:** JSON over HTTPS
- **Authentication:** API key (Bearer token)

> Replace `{{API_BASE_URL}}` throughout this document with your Observability Insights instance's API host, e.g. `https://insights.adobecqms.net/`.

---

## 1. Getting an API key

API keys are personal credentials tied to your account and scoped to a single organization. A key can only read data for tenants that belong to the organization it was created for — it can never see another organization's data.

### Generate a key

1. Sign in to the [Observability Insights dashboard](https://insights.adobecqms.net/).
2. Open your profile menu (top right) → **API Keys**.
3. In the **API Keys** tab, click **Generate key**.
4. Give it a descriptive name (e.g. `CI pipeline`, `Grafana datasource`), choose the organization it should be scoped to, and optionally set an expiration date.
5. Click **Generate key**. Your key is shown **once**, in the format:

   ```
   synx_9pQ2v6f1WYbLZk3n0aRtEo4jXcHsVmDgUiPq7B8l1yc
   ```

   **Copy it immediately and store it somewhere safe** (a secrets manager, CI secret store, etc.) — the dashboard cannot show it to you again. If you lose it, revoke it and generate a new one.

### Manage existing keys

The API Keys section lists every key you've created, including its organization, creation date, expiration, and last-used timestamp. Click the trash icon next to a key to **revoke** it — revocation is immediate and cannot be undone.

### Key security

- Treat an API key exactly like a password. Anyone with the key can read all observability data for every tenant in the organization it's scoped to, until it's revoked or expires.
- Never commit a key to source control or share it in plaintext (chat, email, tickets).
- Rotate keys periodically and revoke any key that's no longer in use.
- If a key is compromised, revoke it immediately from **Org Settings → API Keys** and generate a replacement.

---

## 2. Authenticating requests

Every request to the Public API must include your key in the `Authorization` header:

```
Authorization: Bearer synx_9pQ2v6f1WYbLZk3n0aRtEo4jXcHsVmDgUiPq7B8l1yc
```

Requests without a valid key, or with an expired/revoked key, receive `401 Unauthorized`. Session logins (browser cookies/tokens) are **not** accepted on this API .

---

## 3. Base concepts

### Tenants

Every endpoint requires a `tenant_id` query parameter identifying which tenant's data to read. A key can only query tenants that belong to the organization it was created for; requesting a tenant outside that organization returns `403 Forbidden`. There is no "all tenants" mode on this API — always pass a specific `tenant_id`.

Not sure which `tenant_id` values your key can use? Call [`GET /public/v1/tenants`](#get-publicv1tenants) — it lists exactly the tenants your key is authorized to query.

### Time ranges

Endpoints that accept `from` / `to` parameters take Unix timestamps (seconds), millisecond timestamps, or ISO 8601 datetime strings, e.g.:

```
from=1735689600
from=2025-01-01T00:00:00Z
```

If omitted, most endpoints default to a recent rolling window (see each endpoint below).

### Rate limits

Requests are rate-limited per API key. If you exceed the limit, you'll receive:

```http
HTTP/1.1 429 Too Many Requests
Retry-After: 60

{ "error": "Too Many Requests", "message": "Rate limit of 300 requests/60s exceeded" }
```

Back off and retry after the number of seconds in the `Retry-After` header. Contact support if your use case needs a higher limit.

### Errors

Errors are returned as JSON with an `error` field and, usually, a human-readable `message`:

```json
{ "error": "Bad Request", "message": "tenant_id is required" }
```

| Status                    | Meaning                                                            |
| ------------------------- | ------------------------------------------------------------------ |
| `400 Bad Request`         | Missing or invalid parameter (e.g. no `tenant_id`, bad time range) |
| `401 Unauthorized`        | Missing, invalid, expired, or revoked API key                      |
| `403 Forbidden`           | The key isn't authorized for the requested tenant                  |
| `429 Too Many Requests`   | Rate limit exceeded — see `Retry-After`                            |
| `502 Bad Gateway`         | Upstream query failed — safe to retry                              |
| `503 Service Unavailable` | Data backend temporarily unavailable                               |

---

## 4. Endpoints

### `GET /public/v1/tenants`

Lists the tenant IDs your key is authorized to query. Call this first — every other endpoint requires one of these values as `tenant_id`.

```bash
curl -s "{{API_BASE_URL}}/public/v1/tenants" \
  -H "Authorization: Bearer $Observability_Insights_API_KEY"
```

```json
{ "tenants": ["tenant1", "tenant2"] }
```

### `GET /public/v1/overview`

High-level health KPIs for a tenant over a time window: request volume, error rate, and latency percentiles.

| Param        | Required | Description                                                               |
| ------------ | -------- | ------------------------------------------------------------------------- |
| `tenant_id`  | Yes      | Tenant to query                                                           |
| `from`, `to` | No       | Time range (see [Time ranges](#time-ranges))                              |
| `minutes`    | No       | Shorthand for "last N minutes" if `from`/`to` aren't given (default `15`) |

```bash
curl -s "{{API_BASE_URL}}/public/v1/overview?tenant_id=<tenant_id>&minutes=30" \
  -H "Authorization: Bearer $Observability_Insights_API_KEY"
```

```json
{
  "tenant_id": "<tenant_id>",
  "from": 1735689000,
  "to": 1735690800,
  "total_spans": 48213,
  "errors": 112,
  "error_rate_pct": 0.23,
  "p50_ms": 34,
  "p95_ms": 210,
  "p99_ms": 480,
  "service_count": 12,
  "trace_count": 9021
}
```

### `GET /public/v1/services`

Lists distinct service names reporting for a tenant.

| Param        | Required | Description                                                           |
| ------------ | -------- | --------------------------------------------------------------------- |
| `tenant_id`  | Yes      | Tenant to query                                                       |
| `from`, `to` | No       | Restrict to services seen in this window; defaults to the last 7 days |

```bash
curl -s "{{API_BASE_URL}}/public/v1/services?tenant_id=<tenant_id>" \
  -H "Authorization: Bearer $Observability_Insights_API_KEY"
```

```json
{
  "tenant_id": "<tenant_id>",
  "services": ["checkout-api", "payments-worker", "web-frontend"]
}
```

### `GET /public/v1/traces`

Searches recent traces for a tenant, with optional filters.

| Param             | Required | Description                                       |
| ----------------- | -------- | ------------------------------------------------- |
| `tenant_id`       | Yes      | Tenant to query                                   |
| `from`, `to`      | No       | Time range; defaults to last 24 hours             |
| `limit`           | No       | Max rows to return (1–200, default 100)           |
| `offset`          | No       | Pagination offset (default 0)                     |
| `service`         | No       | Filter by service name                            |
| `app_name`        | No       | Filter by application/instance name               |
| `status`          | No       | Filter by trace status: `ok`, `error`, or `unset` |
| `search`          | No       | Free-text search across span/operation names      |
| `min_duration_ms` | No       | Only traces at or above this duration             |

```bash
curl -s "{{API_BASE_URL}}/public/v1/traces?tenant_id=<tenant_id>&status=error&limit=25" \
  -H "Authorization: Bearer $Observability_Insights_API_KEY"
```

```json
{
  "data": [
    {
      "TraceId": "4bf92f3577b34da6a3ce929d0e0e4736",
      "ServiceName": "checkout-api",
      "DurationMs": 812,
      "StatusCode": "Error",
      "Timestamp": "2026-08-30T09:12:44Z"
    }
  ],
  "rows": 137,
  "limit": 25,
  "offset": 0
}
```

Use `rows` (the total matching count) alongside `limit`/`offset` to page through results.

### `GET /public/v1/traces/:traceId`

Returns the full span waterfall for a single trace.

| Param       | Required | Description                              |
| ----------- | -------- | ---------------------------------------- |
| `tenant_id` | Yes      | Tenant the trace belongs to              |
| `limit`     | No       | Max spans to return (1–500, default 500) |
| `offset`    | No       | Pagination offset for very large traces  |

```bash
curl -s "{{API_BASE_URL}}/public/v1/traces/4bf92f3577b34da6a3ce929d0e0e4736?tenant_id=<tenant_id>" \
  -H "Authorization: Bearer $Observability_Insights_API_KEY"
```

```json
{
  "spans": [
    {
      "SpanId": "00f067aa0ba902b7",
      "Name": "POST /checkout",
      "DurationMs": 812,
      "children": []
    }
  ],
  "totalDurationMs": 812,
  "spanCount": 14,
  "limit": 500,
  "offset": 0
}
```

### `GET /public/v1/metrics`

Returns raw metric data points for a tenant.

| Param                              | Required               | Description                                                                                                                                     |
| ---------------------------------- | ---------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| `tenant_id`                        | Yes                    | Tenant to query                                                                                                                                 |
| `metric`                           | One of `metric`/`like` | Exact metric name                                                                                                                               |
| `like`                             | One of `metric`/`like` | SQL `LIKE` pattern to match multiple metric names                                                                                               |
| `type`                             | No                     | `gauge` (default) or `sum`                                                                                                                      |
| `from`, `to`                       | No                     | Time range; defaults to last 24 hours                                                                                                           |
| `service`                          | No                     | Filter by service name                                                                                                                          |
| `host`                             | No                     | Filter by host name. Required for the infrastructure host metrics below — without it, readings from every host in the tenant are mixed together |
| `attribute_key`, `attribute_value` | No                     | Filter by a specific metric attribute (must be used together)                                                                                   |

```bash
curl -s "{{API_BASE_URL}}/public/v1/metrics?tenant_id=<tenant_id>&metric=jvm.memory.used&type=gauge" \
  -H "Authorization: Bearer $Observability_Insights_API_KEY"
```

```json
{
  "data": [
    {
      "TimeUnix": "2026-08-30T09:00:00Z",
      "MetricName": "jvm.memory.used",
      "Value": 512482816,
      "ServiceName": "checkout-api",
      "host": ""
    }
  ],
  "rows": 1
}
```

#### Infrastructure host metrics

The same endpoint also serves the host-level metrics shown on the Infrastructure dashboard (CPU, memory, load average, disk I/O, network I/O). Use these exact `metric` / `attribute_key` / `attribute_value` combinations, always with a `host`:

| Dashboard widget      | `metric`                              | `attribute_key` | `attribute_value`                                                                           |
| --------------------- | ------------------------------------- | --------------- | ------------------------------------------------------------------------------------------- |
| CPU %                 | `system.cpu.utilization`              | `state`         | `idle` (subtract from 1 for "in use"), or query `user`/`system`/`iowait` separately and sum |
| Memory usage %        | `system.memory.utilization`           | `state`         | `used`                                                                                      |
| Load average (1m)     | `system.cpu.load_average.1m`          | —               | —                                                                                           |
| Disk read I/O         | `system.disk.io` (`type=sum`)         | `direction`     | `read`                                                                                      |
| Disk write I/O        | `system.disk.io` (`type=sum`)         | `direction`     | `write`                                                                                     |
| Disk read operations  | `system.disk.operations` (`type=sum`) | `direction`     | `read`                                                                                      |
| Disk write operations | `system.disk.operations` (`type=sum`) | `direction`     | `write`                                                                                     |
| Network in            | `system.network.io` (`type=sum`)      | `direction`     | `receive`                                                                                   |
| Network out           | `system.network.io` (`type=sum`)      | `direction`     | `transmit`                                                                                  |

```bash
curl -s "{{API_BASE_URL}}/public/v1/metrics?tenant_id=<tenant_id>&metric=system.cpu.utilization&type=gauge&attribute_key=state&attribute_value=idle&host=<host_name>" \
  -H "Authorization: Bearer $Observability_Insights_API_KEY"
```

**Important — disk and network values are raw, ever-increasing counters, not rates.** The dashboard's "bytes/sec" and "operations/sec" charts are computed by taking two consecutive counter readings and dividing by the elapsed time:

```
rate = (value_at_t2 - value_at_t1) / (t2 - t1_in_seconds)
```

### `GET /public/v1/pages`

Top requested content pages (`.html`) per dispatcher instance, ranked by request count. Backed by the `dispatcher.httpd.requests` metric — this endpoint is specific to AEM Dispatcher/CDN-style access logs, not a general page-analytics tool.

| Param        | Required | Description                            |
| ------------ | -------- | -------------------------------------- |
| `tenant_id`  | Yes      | Tenant to query                        |
| `from`, `to` | No       | Time range; defaults to last 24 hours  |
| `limit`      | No       | Max rows to return (1–500, default 50) |

```bash
curl -s "{{API_BASE_URL}}/public/v1/pages?tenant_id=<tenant_id>&limit=50" \
  -H "Authorization: Bearer $Observability_Insights_API_KEY"
```

```json
{
  "tenant_id": "<tenant_id>",
  "from": 1735689000,
  "to": 1735690800,
  "data": [
    {
      "instance": "<instance_name>",
      "domain": "www.abc.com",
      "path": "/join-us/insights.html",
      "full_url": "https://www.abc.com/join-us/insights.html",
      "requests": 7
    }
  ],
  "rows": 1
}
```

---

## 5. What this API does not do

- **No raw SQL access.** All endpoints return curated, purpose-built data shapes — you cannot query the underlying data store directly.
- **No cross-tenant queries.** Every request is scoped to exactly one `tenant_id`.
- **No write access.** The Public API is read-only.

---

## 6. Support

If you run into unexpected errors, or have a use case not covered by these endpoints, contact your Customer Success / Enablement Engineer for further help.
