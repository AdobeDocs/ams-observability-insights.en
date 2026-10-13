---
title: Access and account management
description: Understand how Observability Insights accounts are provisioned, who manages access, and what level of control customer teams have.
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

# Access and account management {#access-and-account-management}

Adobe provisions and manages the Observability Insights account for your AEM Managed Services organization. Customer teams use Adobe IMS to sign in and receive read-only visibility into the monitored data.

## Account ownership model {#account-ownership-model}

- Adobe Managed Services owns the Observability Insights account.
- Customer teams are granted read-only access.
- Administrative changes, provisioning, and access updates are managed through Adobe.

## How users get access {#how-users-get-access}

Access to Observability Insights requires Adobe IMS provisioning.

To request or update access:

1. Contact your Customer Success Engineer (CSE).
2. Provide the user details required for Adobe IMS provisioning.
3. Confirm that the correct organization and access scope have been assigned.

After provisioning is complete, sign in at [insights.adobecqms.net](https://insights.adobecqms.net).

## What users can do {#what-users-can-do}

Customer users can typically:

- View APM dashboards
- View infrastructure dashboards
- Inspect monitored application and host metrics
- Participate in investigations using shared dashboards and traces

## What users cannot do {#what-users-cannot-do}

Customer users should assume that administrative control remains with Adobe Managed Services unless Adobe explicitly documents otherwise.

Common examples include:

- Managing account ownership
- Changing platform-level provisioning
- Altering managed instrumentation behavior

## Information to add later {#information-to-add-later}

Use this section when fuller process detail becomes available:

- Adobe IMS prerequisites
- Expected provisioning turnaround times
- Escalation contacts and paths
- User lifecycle management for joiners and leavers
