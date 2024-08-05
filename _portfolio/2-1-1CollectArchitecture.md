---
title: "Collect Architecture"
excerpt: " Collect the data for analytics: <br/><img src='/images/portfolio/CollectArchitecture.png'>"
collection: portfolio
venue: 'Data Management'
tags:
  - Collect
---

<img src='/images/portfolio/CollectArchitecture.png'>

Source systems are typically designed for transaction processing and cannot curate, transform, or integrate data within themselves. We need to collect data from various sources, such as databases, applications, streams, or external sources, into a data platform for processing. This data is usually collected into cloud storage, which is partitioned into a separate layer of the larger data analytics architecture. This layer is what we call the raw layer and data is stored in it the original source system state.

We will be explain Collect's sub components in this page. I have gone into details of the subcomponents also. Click on individual links to know more.

* [Profile](https://nuneskris.github.io/publication/CollectDataProfiling): Understand the source data's structure and consistency so we can design and plan the data for analytics processing.
* [Capture](https://nuneskris.github.io/publication/Collect-Data-Capture): Define, isolate and filter only the data which is required for analytics processing.
* Extract: Move the data from source system into the target system enviorment. (Usually On-prem/Cloud to Cloud).
* Load: Loading the data into the analytics processing system incrementally.

# [Data Profiling](https://nuneskris.github.io/publication/CollectDataProfiling)
We need to analyze the source data to understand its structure and consistency. I strongly believe, and frequently emphasize to the teams I lead, that data engineering does not deal with large problems but rather a large number of small problems related to data issues. The main objective of profiling data is to scope the size of the data engineering effort, which is a function of the complexity of the structure and inconsistencies of the data.

> Very often, we develop data engineering pipelines based on test data that does not reveal the true extent of the data quality problems we will encounter in production scenarios. This is why it is important to break down data engineering projects into smaller, end-to-end agile cycles where we test the pipelines with production data early, rather than face inevitable surprises from bad data.

> Invest in data profiling tools. I have seen organizations trying to write numerous queries to understand the data. Data engineering is about extracting and transforming data, not writing throwaway queries to understand it.

I have template I have used to much success in multiple occasions in this [page](https://nuneskris.github.io/publication/CollectDataProfiling).

# [Data Capture](https://nuneskris.github.io/publication/Collect-Data-Capture)
Data residing in the source application needs to be captured for an initial load when moving into production, along with updates to the data (deletions, edits, and insertions) during each periodic transfer. These updates are referred to as change data capture, which is the most critical task in the collect component.

However the complexity is in the capturing changes. Below are the key steps to capture data for changes

1. Identify the changed source data within the larger dataset to allow a select on only the changes. This shouhld include all changes (deletes, updates, and inserts).
2. Add metadata on the data files we captures to identify the change capture batch. Implement a watermarking strategy where the last successfully extracted timestamp is recorded. This will be used as the starting point for the next extraction.
3. Add metadata at a row level to indicate whether a row is a delete, update or insert.
4. Test changes capture for deletes, updates and inserts.

I get into details in this page on [Data Capture](https://nuneskris.github.io/publication/Collect-Data-Capture) where I get into recommendations and considerations.

# Data Extraction
Typically we would capture the data changes as files. Connect to the source systems and ingesting data into the data platform periodically (streaming mode or batch mode). We use terms such as ingest or extract for this. A question which I encounter is whether we apply [minor transformation in extraction queries (Pre-ingest) or extract as is from source (Post-ingest)](https://nuneskris.github.io/publication/Collect-Process-Pre-ingest-vs-post-ingest). Though most situtations we need to extrct data as-is, we need to also keep in mind what makes sense.

## [Batch Transfer Data](https://nuneskris.github.io/publication/Collect-Data-Extraction-Batch)
The most common scenario we would encouter is to move data in batches. We need to extract data from source applications using simple queries for each table, outputting the results into flat files. These files are then ingested into analytics platforms in the cloud. I have often observed that pipelines make significant mistakes in this subsystem, especially when transferring data from on-premises to the cloud. This is primarily due to the need for coordination between two teams: the application team, which owns the source application, and the data team, which owns the analytics applications.

## Stream Transfer Data

## [Compresss data](https://nuneskris.github.io/publication/Collect-Data-Extraction-Compress)
Compressing data during ETL ingestion into the cloud offers several benefits, primarily related to performance, cost, and efficiency. Cloud storage costs are typically based on the amount of data stored. Compressing data reduces its size, leading to lower storage costs. Most importantly, compressed data is smaller, which means it can be transferred more quickly over the network. This is particularly important when moving large volumes of data to the cloud, as it speeds up the ingestion process and reduces the load on network resources.

Please take a look at this [Page](https://nuneskris.github.io/publication/Collect-Data-Extraction-Compress) where I get into details on a couple of case studies.

I also did a [demo](https://nuneskris.github.io/talks/Avro-Parquet-CSV-Gzip) to compare compression and speed of CSV, AVRO gzip and Parquet.

## Encrypt Data
Although not a direct benefit of compression, smaller data sizes can make encryption and decryption processes more efficient, enhancing data security during transfer and storage.

# Data Loading

# Related Pages
* https://nuneskris.github.io/publication/CollectDataProfiling
* https://nuneskris.github.io/publication/Collect-Data-Capture
* https://nuneskris.github.io/publication/Collect-Data-Extraction-Compress
* https://nuneskris.github.io/talks/Avro-Parquet-CSV-Gzip
* https://nuneskris.github.io/publication/Collect-Data-Extraction-Batch
