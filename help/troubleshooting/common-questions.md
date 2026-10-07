---
title: FAQ's
description: Common questions and investigation starting points for Observability Insights in AEM Managed Services.
feature: Operations
role: Admin
product_v2:
  - id: fd1f54a9-f50c-467d-8956-cebbaf4f3eb8
    internal-label: Experience Manager
feature_v2:
  - id: 8a70d214-ab7b-58c1-b001-2ed2e5d6303d
    internal-label: Operations
role_v2:
  - id: c66ffd68-0f65-42bb-aa23-b4020f12e0bd
    internal-label: Admin
---

# FAQ's {#faq}

Use this page as a starting point when you are unsure where to begin or need a quick answer during an active investigation.

## Why can't I access Observability Insights? {#cannot-access-observability-insights}

Start with [Access and account management](../get-started/access-and-accounts.md). If your provisioning is incomplete or outdated, contact your Customer Success Engineer (CSE) to request access or an update.

## Why do I see "Loading Permissions" when I try to log in? {#loading-permissions-error}

This typically indicates an issue with your user provisioning. Contact your Customer Success Engineer (CSE), who can work with the relevant teams to resolve the access issue.

## How do I determine whether an issue is application or infrastructure related? {#application-or-infrastructure}

Start with [Application Performance Monitoring](/help/applications.md) to review request rates, error rates, and latency on Author or Publish. If application signals are elevated, use [Hosts](/help/hosts.md) to check whether host-level resource pressure—CPU, memory, disk, or network—explains or compounds what you are seeing.

## What data does Observability Insights actually collect? {#what-data-is-collected}

See [Coverage, environments, and data retention](../get-started/coverage-and-data.md) for monitoring scope, application representation, retention periods, and operational implications.
