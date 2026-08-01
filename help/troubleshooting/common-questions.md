---
title: Troubleshooting and FAQ
description: Common questions and investigation starting points for Synoptryx in AEM Managed Services.
feature: Operations
role: Admin
---

# Troubleshooting and FAQ {#troubleshooting-and-faq}

Use this page as a starting point when you are unsure where to begin or need a quick answer during an active investigation.

## Why can't I access Synoptryx? {#cannot-access-synoptryx}

Start with [Access and account management](../get-started/access-and-accounts.md). If your provisioning is incomplete or outdated, contact your Customer Success Engineer (CSE) to request access or an update.

## Why do I see "Loading Permissions" when I try to log in? {#loading-permissions-error}

This typically indicates an issue with your user provisioning. Contact your Customer Success Engineer (CSE), who can work with the relevant teams to resolve the access issue.

## How do I determine whether an issue is application or infrastructure related? {#application-or-infrastructure}

Start with [Application Performance Monitoring](../application-performance-monitoring.md) to review request rates, error rates, and latency on Author or Publish. If application signals are elevated, use [Infrastructure monitoring](../infrastructure-monitoring.md) to check whether host-level resource pressure—CPU, memory, disk, or network—explains or compounds what you are seeing.

## How should I understand a specific graph or metric? {#understand-graph-or-metric}

Use the dashboard reference pages for panel-by-panel descriptions, metric names, units, and screenshots:

- [APM dashboard reference](../reference/apm-dashboard-reference.md)
- [Infrastructure dashboard reference](../reference/infrastructure-dashboard-reference.md)

## What data does Synoptryx actually collect? {#what-data-is-collected}

See [Coverage, environments, and data retention](../get-started/coverage-and-data.md) for monitoring scope, application representation, retention periods, and operational implications.
