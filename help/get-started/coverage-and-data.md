---
title: Coverage, environments, and data retention
description: See what Synoptryx monitors in AEM Managed Services, how applications are represented, and how long monitoring data is retained.
feature: Operations
role: Admin
---

# Coverage, environments, and data retention {#coverage-environments-and-data-retention}

This page summarizes what data is collected in Synoptryx for AEM Managed Services and how that data is organized.

## Monitoring coverage {#monitoring-coverage}

Adobe monitors:

- AEM Author tiers with the Synoptryx APM Java plug-in
- AEM Publish tiers with the Synoptryx APM Java plug-in
- Hosted servers in the managed topology with the Synoptryx Infrastructure agent

Custom APM and infrastructure monitoring is enabled in both non-production and production Managed Services environments.

## How applications are represented {#how-applications-are-represented}

Each AEM Managed Services environment typically includes:

- One APM application for Author
- One APM application for Publish

All topologies in a Managed Services contract report into one Synoptryx account.

## Data retention {#data-retention}

APM metrics, infrastructure metrics, and related events are retained for up to **30 days**.

## What this means operationally {#what-this-means-operationally}

- Synoptryx is suited to operational analysis, active incidents, and recent trend comparison.
- Historical analysis beyond the retention window should be handled through other reporting or archival processes if required.
- When investigating recurring issues, capture screenshots or exported evidence before the data ages out.

## Add more detail here later {#add-more-detail-here-later}

This structure can be expanded with:

- Environment naming examples
- Production versus non-production coverage specifics
- Agent deployment topology diagrams
- Data availability caveats during maintenance or rollout windows
