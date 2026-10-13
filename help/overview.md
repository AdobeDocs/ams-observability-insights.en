---
title: Monitor your AEM Managed Services environment with Observability Insights
description: Start here to understand what Observability Insights covers in AEM Managed Services, who it is for, and how to navigate the rest of this guide.
feature: Operations
role: Admin
---

# Monitor your AEM Managed Services environment with Observability Insights {#observability-insights-monitoring}

**Observability Insights** provides visibility into application performance, infrastructure health, and service behavior in AEM Managed Services, without requiring a separate monitoring platform.

If you are responsible for service reliability, incident response, or performance analysis, **Observability Insights** helps you move from symptoms to evidence quickly. It combines application telemetry and host-level health signals so customer teams and Adobe can investigate issues from a shared operational view.

## Observability Insights Whitepaper

[Download the Observability Insights Whitepaper](v2-assets/Observability_Insights_Overview.pdf)

## Why teams use Observability Insights? {#why-teams-use-observability-insights}

Use Observability Insights to answer operational questions such as:

- Is the issue affecting Author, Publish, or both?
- Is the problem caused by application behavior, host resource pressure, or a combination of both?
- Which transactions, endpoints, or status groups explain the spike in errors or latency?
- Is the issue isolated to one environment or visible across the broader topology?

Observability Insights is designed for operational analysis of recent behavior. It helps you identify what changed, where it changed, and which signals are most relevant before escalation or corrective action.

## What Observability Insights helps you do? {#what-observability-insights-helps-you-do}

Use Observability Insights to:

- Understand how Author and Publish tiers behave under real traffic.
- Correlate application latency, error rates, and JVM health with host-level signals.
- Confirm whether an issue is isolated to one environment, one tier, or one host.
- Give Adobe Managed Services and your internal teams a shared operational view during investigation.

Observability Insights is included with AEM Managed Services. Adobe provisions and manages the account, instruments supported environments, and exposes the resulting dashboards to your team as read-only operational tooling.

Because Adobe manages the platform setup and instrumentation, you can focus on investigation and interpretation rather than agent deployment, account administration, or dashboard assembly.

## At a glance {#at-a-glance}

As part of AEM Managed Services, you receive:

- **Dedicated Observability Insights account** — Provisioned and overseen by Adobe Managed Services, with read-only access for your team.
- **Deep AEM transaction monitoring** — The Observability Insights APM agent traces meaningful transactions down to method calls (including line numbers), external dependencies, and repository operations.
- **Unified applications and hosts view** — Combine applications and host-level metrics to optimize performance holistically.

## Who this documentation is for {#who-this-documentation-is-for}

This documentation is designed primarily for:

- AEM Managed Services administrators who need visibility into monitored environments
- Operations and support teams handling incidents, trend analysis, and service review
- Customer engineering teams partnering with Adobe Managed Services during investigations
- Stakeholders who need to understand monitoring scope and operational responsibilities

## What Adobe monitors with Observability Insights {#what-we-monitor}

Adobe monitors AEM **Author** and **Publish** tiers with the Observability Insights APM Java plug-in. All hosted servers in your topology are monitored with the Observability Insights Infrastructure agent. Custom APM and Infrastructure monitoring is enabled in both non-production and production Managed Services environments.

![Diagram showing  Observability Insights APM and Infrastructure monitoring across AEM Author, Publish, and hosted servers](v2-assets/login-screen.png)

### Applications in your account {#applications-in-your-account}

Your Observability Insights account is linked to a single Adobe master account and can receive data from multiple applications, including:

- One APM application for the **Author** tier per AEM Managed Services environment
- One APM application for the **Publish** tier per AEM Managed Services environment

Each application has its own license key. All topologies in your Managed Services contract report into one Observability Insights account. APM and Infrastructure metrics and events are retained for up to **30 days**.

## Access your account {#access}

Monitoring data is consolidated in an Observability Insights account that Adobe provisions and manages. Customer users receive **read-only access** to APM and Infrastructure data collected by the agents. Adobe Managed Services retains account ownership and administrative control.

### Prerequisites {#access-prerequisites}

Before you can sign in, confirm the following:

- Your organization has an active **AEM Managed Services** subscription. Observability Insights is included at no additional cost.
- Your Customer Success Engineer (CSE) has provisioned your Adobe IMS account and granted you access to the Observability Insights account for your organization.

>[!NOTE]
>
> **Getting access:** Access to Observability Insights requires Adobe IMS provisioning. Contact your Customer Success Engineer (CSE) to provision and manage user access for your organization.

After the CSE has provisioned the account, sign in at [insights.adobecqms.net](https://insights.adobecqms.net). This URL is the same for all AEM Managed Services customers; your organization's environments and dashboards are scoped to your provisioned account.
