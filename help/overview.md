---
title: Monitor your AEM Managed Services environment with Synoptryx
description: An overview of Synoptryx monitoring on Adobe Experience Manager Managed Services—what Adobe monitors, how your account is set up, and how your team gets access.
feature: Operations
role: Admin
---

# Monitor your AEM Managed Services environment with Synoptryx {#synoptryx-monitoring}

Synoptryx gives your team visibility into application performance, infrastructure health, and end-user experience—without setting up a separate monitoring platform.

>[!NOTE]
>
> A Synoptryx Product Overview Whitepaper is available for the full AEM Managed Services observability and monitoring overview, ideal for sharing with stakeholders or reviewing offline.

## Overview {#overview}

Synoptryx is Adobe's next-generation observability platform designed to provide unified visibility across application performance, infrastructure health, and synthetic monitoring. It enables proactive monitoring of critical business services through a single, integrated experience. Synoptryx combines Application Performance Monitoring (APM), Infrastructure Monitoring, and Synthetic User Journey monitoring to help identify and resolve issues before they impact end users. The platform delivers deep transaction tracing, JVM insights, infrastructure telemetry, and advanced diagnostics for faster root cause analysis. Built on modern observability technologies, it provides scalable and secure monitoring across complex enterprise environments. Synoptryx offers extended data retention, rich dashboards, and intelligent analytics to support operational excellence. Seamless login experience with Adobe IMS ensures secure access and governance. The platform is engineered to improve service reliability, accelerate troubleshooting, and enhance customer experience. As Adobe's strategic observability solution, Synoptryx provides a future-ready foundation for monitoring, automation, and operational insights across managed services environments.

Synoptryx is included with Adobe Experience Manager Managed Services—no separate monitoring platform or license is required. Adobe monitors the availability and performance of your environment as part of our standard offering, and Synoptryx is the dedicated platform your team can use to understand how your Adobe Experience Manager (AEM) application and supporting infrastructure are performing.

This guide explains what is monitored, how your Synoptryx account is set up, and how to navigate the dashboards you use for day-to-day analysis and troubleshooting.

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

## Access and your account {#access}

Monitoring data is consolidated in a Synoptryx account that Adobe provisions and manages. Your team receives **full read-only access** to all APM and Infrastructure metrics collected by the agents. Adobe Managed Services retains ownership and administrative control of the account.

>[!NOTE]
>
> **Getting access:** Access to Synoptryx requires Adobe IMS provisioning. Your Customer Success Engineer (CSE) can provision and manage user access for your organization.

After the CSE has provisioned the account, you can sign in at [synoptryx.adobecqms.net](https://synoptryx.adobecqms.net).

## What's next {#whats-next}

Continue with the monitoring dashboards your team uses day to day:

- [Application performance monitoring (APM)](application-performance-monitoring.md) — Trace AEM transactions, analyze JVM behavior, and inspect external services.
- [Infrastructure monitoring](infrastructure-monitoring.md) — Review host-level system, network, process, and storage metrics.
