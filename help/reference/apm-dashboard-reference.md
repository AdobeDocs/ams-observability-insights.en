---
title: APM dashboard reference
description: Panel-by-panel reference for Synoptryx APM dashboards, including screenshots, metrics, and units.
feature: Operations
role: Admin
---

# APM dashboard reference {#apm-dashboard-reference}

This reference documents the main Synoptryx APM panels used in AEM Managed Services.

## Dashboard navigation

![Dashboard Navigation](../assets/apm/1_opening_screen.png)

The dashboard is organized into expandable sections that group related application performance metrics. Expanding a section reveals one or more charts associated with that category.

## Overview

![Overview](../assets/apm/1.1_apm_overview.png)

### Description

The **Overview** section presents high-level Key Performance Indicators (KPIs) summarizing the current state of the monitored application.

These KPIs provide an at-a-glance summary of application activity, throughput, request success, and overall user experience.

### Metrics

#### Total Requests

Displays the total number of requests processed by the application during the selected time range.

**Metric**

```
total_requests
```

**Unit**

- Count

#### Current Throughput

Displays the current request processing rate.

**Metric**

```
throughput
```

**Unit**

- Requests per second (req/s)

#### Current Error Rate

Displays the percentage of requests resulting in errors.

**Metric**

```
error_rate
```

**Unit**

- Percentage (%)

#### APDEX Score

Displays the Application Performance Index (APDEX), a standardized measurement of end-user satisfaction based on application response times.

The configured threshold is displayed within the widget.

**Metric**

```
apdex_score
```

**Unit**

- Score (0.0 – 1.0)

## RED Metrics

The RED methodology measures three primary characteristics of an application:

- **Rate**
- **Errors**
- **Duration**

### Request Rate

![Request Rate](../assets/apm/2_red_metrics_request_rate.png)

#### Description

Displays the number of application requests received over time.

This graph represents request throughput using a time-series visualization.

#### Metric

```
req_min
```

#### Unit

- Requests per minute (req/m)

#### Displayed Information

- Time-series request rate
- Historical request activity
- Request rate trend
- Metric legend

### Error Rate

![Error Rate](../assets/apm/3_error_rate.png)

#### Description

Displays the percentage of requests that resulted in errors.

The chart compares historical and current error percentages.

#### Metrics

```
error_pct (now)
error_pct (1h ago)
```

#### Unit

- Percentage (%)

#### Displayed Information

- Current error percentage
- Historical comparison
- Mean values
- Time-series trend

### Request Duration

![Request Duration](../assets/apm/4_request_duration_p50_p95.png)

#### Description

Displays request latency across multiple response time percentiles.

The graph simultaneously plots percentile latency measurements collected during the selected observation period.

#### Metrics

```
P50
P75
P90
```

#### Units

- Milliseconds (ms)
- Seconds (s)

Units automatically scale depending on response duration.

#### Displayed Statistics

For every percentile:

- Mean
- Last
- Maximum

#### Percentile Definitions

| Metric | Description                   |
| ------ | ----------------------------- |
| P50    | 50th percentile response time |
| P75    | 75th percentile response time |
| P90    | 90th percentile response time |

## Traffic

### Requests by HTTP Status Code

![Requests by Status Code](../assets/apm/5_requests_by_status_code.png)

#### Description

Displays request throughput grouped by HTTP response status code.

Each status code is plotted independently over time.

#### Metrics

Common metrics include:

```
req_s 200
req_s 300
req_s 400
req_s 500
```

depending on application activity.

#### Unit

- Requests per second (req/s)

#### Displayed Information

- Throughput by HTTP status
- Mean throughput
- Latest throughput
- Maximum throughput
- Time-series activity

### Request Rate by Endpoint

![Request Rate by Endpoint](../assets/apm/6_request_rate_by_end_point.png)

#### Description

Displays the highest-traffic application endpoints ranked by request rate.

Each endpoint is displayed as a horizontal bar representing request volume.

#### Metric

```
endpoint_request_rate
```

#### Unit

- Requests per minute (req/m)

#### Displayed Information

- Endpoint path
- Request rate
- Ranked endpoint list
- Relative request volume

## Latency and performance

### Response Time — P95 vs 1 Hour

![Response Time P95](../assets/apm/7_response_time_p95_1h.png)

#### Description

Displays a comparison of the current P95 response time against the P95 response time recorded one hour earlier.

Both datasets are displayed on the same time-series graph.

#### Metrics

```
P95 (Current)
P95 (1 Hour Ago)
```

#### Units

- Milliseconds (ms)
- Seconds (s)

#### Displayed Statistics

- Mean
- Last
- Maximum

### APDEX Score Over Time

![APDEX](../assets/apm/8_apdex_score_overtime.png)

#### Description

Displays the Application Performance Index as a continuous time-series.

The graph visualizes APDEX values throughout the selected monitoring interval.

#### Metric

```
APDEX Score
```

#### Unit

- Score (0.0–1.0)

#### Displayed Statistics

- Mean
- Last
- Maximum

### Throughput vs P95 Latency

![Throughput vs Latency](../assets/apm/9_throughput_vs_p95latency.png)

#### Description

Displays request throughput and P95 response latency on the same timeline.

The graph enables simultaneous visualization of traffic volume and response latency.

#### Metrics

```
Throughput
P95 Latency
```

#### Units

| Metric      | Unit         |
| ----------- | ------------ |
| Throughput  | Requests/sec |
| P95 Latency | Milliseconds |

#### Displayed Information

- Time-series throughput
- Time-series latency
- Dual metric comparison

## Error details

### Error Rate % by Status Group

![Error Rate by Status Group](../assets/apm/10_error_rate_pct_by_status_group.png)

#### Description

Displays application error percentages grouped by HTTP response class.

Separate series are plotted for each response category.

#### Metrics

Common groups include:

```
2xx
3xx
4xx
5xx
Combined Error Trend
```

depending on observed traffic.

#### Unit

- Percentage (%)

#### Displayed Information

- Error percentage by response class
- Mean error percentage
- Time-series trend

### Error Ratio Trend — Now vs 1 Hour Ago

![Error Ratio 1 Hour](../assets/apm/11_error_ratio_trend_1h.png)

#### Description

Displays the current application error ratio alongside the error ratio recorded one hour earlier.

#### Metrics

```
Current Error Ratio
1 Hour Error Ratio
```

#### Unit

- Percentage (%)

#### Displayed Information

- Current trend
- Historical comparison
- Time-series visualization

### Error Ratio Trend — Now vs 6 Hours Ago

![Error Ratio 6 Hours](../assets/apm/12_error_ratio_trend_6h.png)

#### Description

Displays the current application error ratio alongside the error ratio recorded six hours earlier.

#### Metrics

```
Current Error Ratio
6 Hour Error Ratio
```

#### Unit

- Percentage (%)

#### Displayed Information

- Current error ratio
- Historical comparison
- Time-series visualization

## Summary of dashboard metrics

| Dashboard                  | Primary Metrics                               |
| -------------------------- | --------------------------------------------- |
| Overview                   | Total Requests, Throughput, Error Rate, APDEX |
| Request Rate               | Requests per Minute                           |
| Error Rate                 | Error Percentage                              |
| Request Duration           | P50, P75, P90 Latency                         |
| Requests by Status Code    | HTTP Status Throughput                        |
| Request Rate by Endpoint   | Endpoint Request Volume                       |
| Response Time Comparison   | Current vs Historical P95                     |
| APDEX Score                | User Satisfaction Index                       |
| Throughput vs Latency      | Request Throughput and P95 Latency            |
| Error Rate by Status Group | HTTP Status Group Error Percentage            |
| Error Ratio Trends         | Current vs Historical Error Ratio             |
