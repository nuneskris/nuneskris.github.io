---
title: "Collect - Extract and Load Patterns"
excerpt: "Transfer data from source to taget <br/><img src='/images/portfolio/ExtractLoad.png'>"
collection: portfolio
venue: 'Processing'
date: 2024-05-01
tags:
  - Collect
---
<img src='/images/portfolio/ExtractLoad.png'>

# 1. Full Load
<img width="612" alt="image" src="/images/portfolio/FullLoadPattern.png">

Every time the process runs, the entire data set is collected from the source into the target analytics system. 
* ***Initial Data Load & Data Reconciliation***: This is usually pattern we would need even if we adopt an incremental load during initial loads. Also when we want to rebase or reconcile our data we would need to run have an ability to run a Full Load.
* ***Operational Reporting***: I have used fully loads for operational reporting when there is limited capabilities within the source application. Ideally the data would not require complex transformations and integration with data across the enterprise.
* ***Backup, Recovery, Compliance & Audit***: An abvious reason this pattern is used is when snapshots of the data is completed ingested into Cloud storage for Backup, Recovery, Compliance & Audit. 
* ***Downstream Delta Processing***: This pattern is also used as part of a larger data processing pipeline, where transformation components deal with processing deltas.

# 2. Append Only Load
<img width="612" alt="image" src="/images/portfolio/AppendLoadPattern.png">

Most often datasets get too big for full load, and collecting data changes will be able to scale. So, any new records in the source since the last sync are appended to the target database without modifying the data already existing. If a record is modified at the source, the new version is appended as a duplicate of the existing row. The dataset in the target is continously increments with the new data appended to it. The load's main functionality is to be able to append the data to the end (logically) of the previous sync
* ***Immutable Target Database***: This is another common dataset in the world of big data where the target databases are immutable and we can only add new data. Most of today's landing zone uses this pattern where focus is on the extraction query offloading the data in a scallable cloud storage filesystems.
* ***Log File Processing***: 
* ***Data Science Layer***: Many of the data science layers (Raw) happens in this layer where raw data is ingested into this layer.
* ***Limited Load Processing Capabilities***: 

# 3. Delta Load
<img width="612" alt="image" src="/images/portfolio/DeltaPattern.png">

# 4. Upsert
<img width="612" alt="image" src="/images/portfolio/UpsertIcon.png">

# 5. SCD2
<img width="612" alt="image" src="/images/portfolio/SCD2Pattern.png">
