---
title: Investigate infrastructure issues
description: Use host-level infrastructure dashboards to determine whether incidents are related to CPU, memory, storage, network, or I/O contention.
feature: Operations
role: Admin
---

# Investigate infrastructure issues {#investigate-infrastructure-issues}

Use this workflow when the issue appears to involve host resource pressure, uneven instance behavior, or platform conditions that may affect application performance.

## Investigation workflow {#investigation-workflow}

1. Confirm the affected environment and host or host group.
2. Review CPU utilization, load average, and memory usage for obvious saturation.
3. Check CPU I/O wait and disk throughput if latency rises without a clear CPU spike.
4. Review network throughput during the same interval to detect traffic shifts or transfer bottlenecks.
5. Check storage usage and filesystem utilization for capacity-related risk.
6. Compare with APM behavior to determine whether the problem is app-driven, infrastructure-driven, or mixed.

## Questions to answer during triage {#questions-to-answer-during-triage}

- Is the issue isolated to one host or visible across the environment?
- Are CPU, memory, or disk signals elevated during the same window as the incident?
- Is storage growth trending toward a threshold that could affect operations?
- Do infrastructure symptoms explain the application behavior, or do they only appear as a downstream effect?

## Evidence to capture {#evidence-to-capture}

- Affected environment and hosts
- Time window of the issue
- CPU, memory, disk, and network screenshots
- Whether the anomaly is isolated or systemic
- Related application symptoms from APM

## Supporting reference {#supporting-reference}

- [Infrastructure monitoring](../infrastructure-monitoring.md)
- [Infrastructure dashboard reference](../reference/infrastructure-dashboard-reference.md)
- [Application Performance Monitoring](../application-performance-monitoring.md)

## Add more workflow detail here later {#add-more-workflow-detail-here-later}

This page is ready to expand with:

- Capacity and saturation thresholds
- Disk growth investigation patterns
- Host comparison techniques
- Escalation criteria for Adobe Managed Services
