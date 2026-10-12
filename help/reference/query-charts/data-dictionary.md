## Data dictionary {#data-dictionary}

This is every entity and attribute you can query. The live catalog, with descriptions, is also in the **Query guide** panel in Explore and in the Add widget dialog.

### Trace entities

| Entity        | Contains                                                  | Fixed scope                  | Rollups                                    |
| ------------- | --------------------------------------------------------- | ---------------------------- | ------------------------------------------ |
| `Transaction` | Inbound server requests                                   | `kind = Server`              | 1m / 5m / 1h ([Performance](#performance)) |
| `AppRequest`  | Inbound server **and** message-consumer requests          | `kind IN (Server, Consumer)` | —                                          |
| `Span`        | Every span (server, client, producer, consumer, internal) | —                            | —                                          |

<details open>
<summary><strong>Trace attributes</strong> (identical on all three entities)</summary>

| Attribute                                           | Type        | Description                                                                                     |
| --------------------------------------------------- | ----------- | ----------------------------------------------------------------------------------------------- |
| `timestamp`                                         | timestamp   | Span start time                                                                                 |
| `traceId`, `spanId`, `parentSpanId`                 | string      | Identifiers (root spans have `parentSpanId = ''`)                                               |
| `name`                                              | string      | Operation or transaction name                                                                   |
| `kind`                                              | string      | `Server`, `Client`, `Producer`, `Consumer`, `Internal`                                          |
| `service`                                           | string      | Emitting service name                                                                           |
| `serviceKey`                                        | string      | Effective service identity (falls back to `appName` when `service` is blank or a coarse parent) |
| `appName`                                           | string      | Observability Insights application name                                                         |
| `appInstance`                                       | string      | Per-instance identity: author/publish host prefix, then app name, then service                  |
| `hostTierPrefix`                                    | string      | Host name truncated at its `author` / `publish` / `dispatcher` role suffix                      |
| `duration`                                          | number (ms) | Span duration in milliseconds                                                                   |
| `status`                                            | string      | `Ok`, `Error`, `Unset`                                                                          |
| `statusMessage`                                     | string      | Status description                                                                              |
| `host`                                              | string      | Host name                                                                                       |
| `environment`                                       | string      | Deployment environment                                                                          |
| `region`                                            | string      | Cloud region                                                                                    |
| `cloud.provider`                                    | string      | AWS / Azure / GCP                                                                               |
| `http.method`                                       | string      | `GET`, `POST`, …                                                                                |
| `http.statusCode`                                   | number      | HTTP response status                                                                            |
| `http.route`                                        | string      | Normalized route template                                                                       |
| `url`                                               | string      | Full URL                                                                                        |
| `urlPath`                                           | string      | Path part of the URL (`/content/site/en.html`)                                                  |
| `externalDomain`                                    | string      | Domain of an outbound URL (use on client spans)                                                 |
| `peer.service`                                      | string      | Downstream service called                                                                       |
| `db.system`, `db.operation`, `db.namespace`         | string      | Database technology, operation and schema                                                       |
| `messaging.system`, `messaging.destination`         | string      | Kafka, RabbitMQ, … and topic or queue                                                           |
| `rpc.system`, `rpc.method`                          | string      | gRPC and similar                                                                                |
| `error.type`                                        | string      | Exception or error type                                                                         |
| `sling.script`, `sling.servlet`, `sling.requestUri` | string      | AEM Sling details                                                                               |
| `jsp.path`                                          | string      | AEM JSP executed                                                                                |
| `jcr.operation`, `jcr.path`                         | string      | AEM JCR repository operation and path                                                           |
| `graphql.operation`                                 | string      | GraphQL operation name                                                                          |

</details>

### Generic metric entities

| Entity        | Contains                                           |
| ------------- | -------------------------------------------------- |
| `MetricGauge` | Every point-in-time OpenTelemetry gauge sample     |
| `MetricSum`   | Every cumulative or delta OpenTelemetry sum sample |

Use them for **discovery**, or for metrics that don't have a curated entity: `WHERE metric = 'jvm.gc.duration'`.

<details open>
<summary><strong>Metric attributes</strong> (shared by all metric entities)</summary>

| Attribute                                                               | Type      | Description                                             |
| ----------------------------------------------------------------------- | --------- | ------------------------------------------------------- |
| `timestamp`                                                             | timestamp | Sample time                                             |
| `metric`                                                                | string    | OpenTelemetry metric name                               |
| `value`                                                                 | number    | Sample value (converted on curated entities; see below) |
| `unit`                                                                  | string    | Metric unit                                             |
| `service`, `appName`, `host`, `environment`, `region`, `cloud.provider` | string    | Resource identity                                       |
| `state`                                                                 | string    | CPU mode or memory state                                |
| `cpu`, `device`, `direction`, `mountpoint`                              | string    | Per CPU, per device, I/O direction, filesystem          |
| `pool`                                                                  | string    | JVM memory pool or connection pool                      |
| `topic`                                                                 | string    | Kafka topic                                             |
| `process.pid`, `process.name`, `process.owner`                          | string    | Process identity                                        |

</details>

### Curated metric entities

Each curated entity wraps one OpenTelemetry metric with the right filter and unit conversion, so `value` is already meaningful. **Counter** entities (✔) hold cumulative totals. Query them with `rate(value)` and `TIMESERIES`.

| Entity                                                                         | `value` means                       | Unit      | Counter | Notes                                        |
| ------------------------------------------------------------------------------ | ----------------------------------- | --------- | ------- | -------------------------------------------- |
| `CpuUtilization`                                                               | CPU busy                            | %         |         | 1-minute rollup for `average(value)` by host |
| `CpuUserPercent` / `CpuSystemPercent` / `CpuIowaitPercent` / `CpuStealPercent` | CPU time by mode                    | %         |         |                                              |
| `MemoryUtilization`                                                            | Memory used                         | %         |         | 1-minute rollup for `average(value)`         |
| `LoadAverage1m` / `LoadAverage5m` / `LoadAverage15m`                           | Load average                        | —         |         |                                              |
| `LoadAverageWithFallback`                                                      | 5m load, falling back to 1m         | —         |         |                                              |
| `DiskUtilization`                                                              | Disk used per mount                 | %         |         | Facet by `mountpoint`                        |
| `DiskFreeBytes`                                                                | Root filesystem free                | bytes     |         |                                              |
| `DiskReadRate` / `DiskWriteRate`                                               | Disk throughput                     | bytes/s   | ✔       |                                              |
| `DiskReadOpsRate` / `DiskWriteOpsRate`                                         | Disk IOPS                           | ops/s     | ✔       |                                              |
| `NetworkReceiveRate` / `NetworkTransmitRate`                                   | eth0 throughput                     | bytes/s   | ✔       |                                              |
| `NetworkReceiveRateAllDevices` / `NetworkTransmitRateAllDevices`               | All devices                         | bytes/s   | ✔       |                                              |
| `NetworkIoRate`                                                                | All directions and devices combined | bytes/s   | ✔       |                                              |
| `NetworkReceiveErrorRate` / `NetworkTransmitErrorRate`                         | Network errors                      | errors/s  | ✔       |                                              |
| `NetworkReceivePacketRate` / `NetworkTransmitPacketRate`                       | Packets                             | packets/s | ✔       |                                              |
| `ProcessCpuUtilization`                                                        | Per-process CPU                     | %         |         | Facet by `process.name`                      |
| `ProcessMemoryUsage`                                                           | Per-process memory                  | MB        |         |                                              |
| `JvmHeapUsed` / `JvmHeapLimit`                                                 | JVM heap                            | MB        |         | Reported per memory pool                     |
| `JvmThreadCount`                                                               | JVM threads                         | —         |         |                                              |
| `JvmCpuRecentUtilization`                                                      | JVM process CPU                     | %         |         | 1-minute rollup                              |
| `DispatcherRequestRate`                                                        | AEM dispatcher HTTP requests        | req/s     | ✔       |                                              |

> [!NOTE]
> JVM memory metrics are reported **per memory pool**, so `average(value)` on `JvmHeapUsed` / `JvmHeapLimit` averages across pools. Use `FACET host, pool` for an exact per-pool view.

### `SpanBreakdown`: pre-aggregated transaction composition

`SpanBreakdown` holds the per-minute child-span composition of each transaction. It's computed at ingest, so it's cheap to query.

| Attribute       | Type      | Description                                                            |
| --------------- | --------- | ---------------------------------------------------------------------- |
| `timestamp`     | timestamp | One-minute bucket                                                      |
| `service`       | string    | Service                                                                |
| `rootOperation` | string    | Parent transaction name                                                |
| `segment`       | string    | Child span name                                                        |
| `callCount`     | number    | Calls of this segment in the bucket                                    |
| `avgDurationMs` | number    | Average segment duration in the bucket (set the unit to `ms` manually) |
