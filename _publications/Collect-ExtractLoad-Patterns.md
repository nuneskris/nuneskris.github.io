---
title: "Collect - Extract and Load Patterns"
collection: publications
permalink: /publication/Collect-ExtractLoad-Patterns
excerpt: 'Collect Patterns'
venue: 'Processing'
date: 2024-05-01
tags:
  - Collect
---

# 1. Full Load
<img width="612" alt="image" src="/images/portfolio/FullLoadPattern.png">

Every time the process runs, the entire data set is collected from the source into the target analytics system. 
* ***Initial Data Load & Data Reconciliation***: This is usually pattern we would need even if we adopt an incremental load during initial loads. Also when we want to rebase or reconcile our data we would need to run have an ability to run a Full Load.
* ***Operational Reporting***: I have used fully loads for operational reporting when there is limited capabilities within the source application. Ideally the data would not require complex transformations and integration with data across the enterprise.
* ***Backup, Recovery, Compliance & Audit***: An abvious reason this pattern is used is when snapshots of the data is completed ingested into Cloud storage for Backup, Recovery, Compliance & Audit. 
* ***Downstream Delta Processing***: This pattern is also used as part of a larger data processing pipeline, where transformation components deal with processing deltas.

# 2. Append Only Load
<img width="612" alt="image" src="/images/portfolio/AppendLoadPattern.png">

