---
title: Infrastructure monitoring with Synoptryx
description: Learn when to use the infrastructure dashboards, what signals to review first, and where to find the full host metric reference.
feature: Operations
role: Admin
---

# Infrastructure monitoring with Synoptryx {#infrastructure-monitoring}

Use the infrastructure dashboards when you need to determine whether an issue is related to host capacity, storage pressure, network throughput, or operating system resource contention.

## What infrastructure monitoring helps you answer {#what-infrastructure-monitoring-helps-you-answer}

Infrastructure monitoring is most useful when you need to answer questions such as:

- Is the application slowdown accompanied by CPU, memory, or I/O pressure?
- Is one host behaving differently from others in the same environment?
- Are network or disk patterns changing during the same interval as a customer-facing incident?
- Are storage utilization trends pointing to an upcoming capacity problem?

## Dashboard overview {#dashboard-overview}

The host dashboard provides real-time visibility into compute, memory, storage, and network conditions across monitored infrastructure. Use it alongside APM when you want to distinguish between application-level regressions and underlying capacity constraints.

The dashboard includes these major panels:

- Host CPU Utilization
- Host Disk I/O
- Host Network I/O
- CPU I/O Wait
- Storage Usage
- Disk Usage
- Host CPU Load Average
- Host Memory Usage

## Suggested investigation flow {#suggested-investigation-flow}

For most incidents, review the host dashboard in this order:

1. Check CPU utilization, load average, and memory usage for obvious saturation.
2. Review CPU I/O wait and disk throughput if response times increase without a matching CPU spike.
3. Compare network throughput with application traffic to identify load-related shifts.
4. Check storage usage and filesystem-level disk usage for persistent capacity risk.
5. Compare multiple hosts to see whether the issue is localized or systemic.

## What to review first {#what-to-review-first}

- **CPU and memory** when an application appears slow or unstable across a broader time range.
- **Disk I/O and CPU I/O wait** when requests stall or queue unexpectedly.
- **Network I/O** when traffic characteristics change or downstream dependencies are suspected.
- **Storage usage** when incidents involve deployment failures, indexing pressure, or long-term capacity concerns.

## Detailed dashboard reference {#detailed-dashboard-reference}

For panel-by-panel descriptions, screenshots, and metric definitions, see [Infrastructure dashboard reference](reference/infrastructure-dashboard-reference.md).

## Related content {#related-content}

- [Investigate infrastructure issues](use-cases/investigate-infrastructure-issues.md)
- [Application Performance Monitoring (APM)](application-performance-monitoring.md)
- [Infrastructure dashboard reference](reference/infrastructure-dashboard-reference.md)
