---
title: Monitor your AEM Managed Services environment with Synoptryx
description: Start here to understand what Synoptryx covers in AEM Managed Services, who it is for, and how to navigate the rest of this guide.
feature: Operations
role: Admin
---

# Monitor your AEM Managed Services environment with Synoptryx {#synoptryx-monitoring}

Synoptryx provides visibility into application performance, infrastructure health, and service behavior in Adobe Experience Manager Managed Services, without requiring a separate monitoring platform.

If you are responsible for service reliability, incident response, or performance analysis, Synoptryx helps you move from symptoms to evidence quickly. It combines application telemetry and host-level health signals so customer teams and Adobe Managed Services can investigate issues from a shared operational view.

> [!VIDEO](https://video.tv.adobe.com/v/3496609

> [!NOTE]
>
> A Synoptryx Product Overview Whitepaper is available for the full AEM Managed Services observability and monitoring overview, ideal for sharing with stakeholders or reviewing offline.

## Why teams use Synoptryx {#why-teams-use-synoptryx}

Use Synoptryx to answer operational questions such as:

- Is the issue affecting Author, Publish, or both?
- Is the problem caused by application behavior, host resource pressure, or a combination of both?
- Which transactions, endpoints, or status groups explain the spike in errors or latency?
- Is the issue isolated to one environment or visible across the broader topology?

Synoptryx is designed for operational analysis of recent behavior. It helps you identify what changed, where it changed, and which signals are most relevant before escalation or corrective action.

## What Synoptryx helps you do {#what-synoptryx-helps-you-do}

Use Synoptryx to:

- Understand how Author and Publish tiers behave under real traffic.
- Correlate application latency, error rates, and JVM health with host-level signals.
- Confirm whether an issue is isolated to one environment, one tier, or one host.
- Give Adobe Managed Services and your internal teams a shared operational view during investigation.

Synoptryx is included with AEM Managed Services. Adobe provisions and manages the account, instruments supported environments, and exposes the resulting dashboards to your team as read-only operational tooling.

Because Adobe manages the platform setup and instrumentation, you can focus on investigation and interpretation rather than agent deployment, account administration, or dashboard assembly.

This guide is organized to help readers reach the right level of detail quickly:

- New users can start with onboarding, access, and coverage information.
- Operators can go directly to the task-oriented monitoring guides.
- Deep-dive investigations can use the dashboard reference for specific metrics and panels.

## Who this documentation is for {#who-this-documentation-is-for}

This documentation is designed primarily for:

- AEM Managed Services administrators who need visibility into monitored environments
- Operations and support teams handling incidents, trend analysis, and service review
- Customer engineering teams partnering with Adobe Managed Services during investigations
- Stakeholders who need to understand monitoring scope and operational responsibilities

## At a glance {#at-a-glance}

As part of AEM Managed Services, you receive:

- **Dedicated Synoptryx account** — Provisioned and overseen by Adobe Managed Services, with read-only access for your team.
- **Deep AEM transaction monitoring** — The Synoptryx APM agent traces meaningful transactions down to method calls (including line numbers), external dependencies, and repository operations.
- **Unified application and infrastructure view** — Combine APM and host-level metrics to optimize performance holistically.

## What Adobe monitors with Synoptryx {#what-we-monitor}

Adobe monitors AEM **Author** and **Publish** tiers with the Synoptryx APM Java plug-in. All hosted servers in your topology are monitored with the Synoptryx Infrastructure agent. Custom APM and Infrastructure monitoring is enabled in both non-production and production Managed Services environments.

![Diagram showing Synoptryx APM and Infrastructure monitoring across AEM Author, Publish, and hosted servers](assets/image6.png)

### Applications in your account {#applications-in-your-account}

Your Synoptryx account is linked to a single Adobe master account and can receive data from multiple applications, including:

- One APM application for the **Author** tier per AEM Managed Services environment
- One APM application for the **Publish** tier per AEM Managed Services environment

Each application has its own license key. All topologies in your Managed Services contract report into one Synoptryx account. APM and Infrastructure metrics and events are retained for up to **30 days**.

## Core monitoring experiences {#core-monitoring-experiences}

Synoptryx documentation is organized around the monitoring experiences used most often.

### Application Performance Monitoring {#application-performance-monitoring-experience}

Use APM when you need to understand request throughput, error rates, latency, JVM behavior, endpoint hotspots, and trace-level execution details for AEM applications.

APM is typically the starting point when the issue involves customer-facing application behavior, such as slow pages, unstable transactions, or rising error rates.

### Infrastructure monitoring {#infrastructure-monitoring-experience}

Use infrastructure dashboards when you need to determine whether the underlying hosts show CPU, memory, network, disk, or storage signals that explain the application behavior.

Infrastructure monitoring is especially useful when you need to distinguish between application-level behavior and environment-level resource constraints.

### Reference and troubleshooting {#reference-and-troubleshooting-experience}

Use the reference material when you need exact panel definitions, metric names, units, and screenshots. Use troubleshooting content for common entry points, FAQ-style guidance, and support-oriented workflows.

## Choose your path {#choose-your-path}

Use the section that matches your immediate goal:

- [Get started with Synoptryx](get-started/onboarding.md) for first-time orientation.
- [Access and account management](get-started/access-and-accounts.md) if you need Adobe IMS access or want to understand user roles.
- [Application Performance Monitoring](application-performance-monitoring.md) to learn how to work with Author and Publish application dashboards.
- [Infrastructure monitoring](infrastructure-monitoring.md) to interpret host-level health and capacity signals.
- [APM dashboard reference](reference/apm-dashboard-reference.md) and [Infrastructure dashboard reference](reference/infrastructure-dashboard-reference.md) for panel-level metric details.

## Recommended starting points {#recommended-starting-points}

If you are not sure where to begin, use one of these paths:

### I am new to Synoptryx

1. Read [Get started with Synoptryx](get-started/onboarding.md).
2. Review [Access and account management](get-started/access-and-accounts.md).
3. Review [Coverage, environments, and data retention](get-started/coverage-and-data.md).

### I am investigating a current issue

1. Start with [Application Performance Monitoring](application-performance-monitoring.md) if the symptom is application-facing.
2. Use [Infrastructure monitoring](infrastructure-monitoring.md) if you suspect host-level pressure or need to confirm capacity impact.
3. Use [Investigate application issues](use-cases/investigate-application-issues.md) or [Investigate infrastructure issues](use-cases/investigate-infrastructure-issues.md) for step-by-step investigation flows.

### I need detailed metric interpretation

Go directly to [APM dashboard reference](reference/apm-dashboard-reference.md), [Infrastructure dashboard reference](reference/infrastructure-dashboard-reference.md), and [Metrics, coverage, and retention](reference/metrics-and-retention.md).

## Access and your account {#access}

Monitoring data is consolidated in a Synoptryx account that Adobe provisions and manages. Your team receives **full read-only access** to all APM and Infrastructure metrics collected by the agents. Adobe Managed Services retains ownership and administrative control of the account.
Monitoring data is consolidated in a Synoptryx account that Adobe provisions and manages. Customer users receive **read-only access** to APM and Infrastructure data collected by the agents. Adobe Managed Services retains account ownership and administrative control.

> [!NOTE]
>
> **Getting access:** Access to Synoptryx requires Adobe IMS provisioning. Your Customer Success Engineer (CSE) can provision and manage user access for your organization.

After the CSE has provisioned the account, you can sign in at [synoptryx.adobecqms.net](https://synoptryx.adobecqms.net).

## How this guide is structured {#how-this-guide-is-structured}

The documentation is organized into four layers:

- **Overview and get started** for orientation, access, and monitoring scope
- **Use Synoptryx** for product-area guidance and investigation workflows
- **Reference** for detailed panel-by-panel documentation
- **Troubleshooting and FAQ** for quick answers and future runbook expansion

## What's next {#whats-next}

Continue with the monitoring dashboards your team uses day to day:

- [Get started with Synoptryx](get-started/onboarding.md) — Understand audience, workflows, and how the documentation is organized.
- [Application performance monitoring (APM)](application-performance-monitoring.md) — Trace AEM transactions, analyze JVM behavior, and inspect external services.
- [Infrastructure monitoring](infrastructure-monitoring.md) — Review host-level system, network, process, and storage metrics.
- [Troubleshooting and FAQ](troubleshooting/common-questions.md) — Start from common questions and support-oriented paths when you are unsure where to begin.
