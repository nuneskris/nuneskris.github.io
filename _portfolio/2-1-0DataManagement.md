---
title: "Data Management"
excerpt: "Making Data Useful and Integrated: <br/><img src='/images/portfolio/DataManagement.png'>"
collection: portfolio
venue: 'Engineering'
tags:
  - Engineering
  - Management
---
<img width="612" alt="image" src="/images/portfolio/DataManagement.png">

We want transform data into information which can lead to business insights by taking data from multiple business functions to make useful and integrated. 
Now we are getting into the realms where many architecture solutions come into play to realize this. Irrespective of the architecture solution, there are key management capabilities each of them try to solve. They are

# Data Processing Capabilities.

## Collect
Source systems are typically designed for transaction processing and cannot curate, transform, or integrate data within themselves. We need to collect data from various sources, such as databases, applications, streams, or external sources, into a data platform for processing. This data is usually collected into cloud storage, which is partitioned into a separate layer of the larger data analytics architecture. This layer is what we call the raw layer and data is stored in it the original source system state.

### Data Profiling
We need to analyze the source data to understand its structure and consistency. I strongly believe, and frequently emphasize to the teams I lead, that data engineering does not deal with large problems but rather a large number of small problems related to data issues. The main objective of profiling data is to scope the size of the data engineering effort, which is a function of the complexity of the structure and inconsistencies of the data.

> Very often, we develop data engineering pipelines based on test data that does not reveal the true extent of the data quality problems we will encounter in production scenarios. This is why it is important to break down data engineering projects into smaller, end-to-end agile cycles where we test the pipelines with production data early, rather than face inevitable surprises from bad data.

> Invest in data profiling tools. I have seen organizations trying to write numerous queries to understand the data. Data engineering is about extracting and transforming data, not writing throwaway queries to understand it.

I have template I have used to much success in multiple occasions in this [page](https://nuneskris.github.io/publication/CollectDataProfiling).

### Data Capture
Data residing in the source application needs to be captured for an initial load when moving into production, along with updates to the data (deletions, edits, and insertions) during each periodic transfer. These updates are referred to as change data capture, which is the most critical task in the collect component.

However the complexity is in the capturing changes. Below are the key steps to capture data for changes

1. Identify the changed source data within the larger dataset to allow a select on only the changes. This shouhld include all changes (deletes, updates, and inserts).
2. Add metadata on the data files we captures to identify the change capture batch. Implement a watermarking strategy where the last successfully extracted timestamp is recorded. This will be used as the starting point for the next extraction.
3. Add metadata at a row level to indicate whether a row is a delete, update or insert.
4. Test changes capture for deletes, updates and inserts.

I get into details in this page on [Data Capture](https://nuneskris.github.io/publication/Collect-Data-Capture) where I get into recommendations and considerations.

### Data Extraction
Typically we would capture the data changes as files. Connect to the source systems and ingesting data into the data platform periodically (streaming mode or batch mode). We use terms such as ingest or extract for this. Below are the the most important considerations.

### Batch Transfer Data

### Compresss data
Compressing data during ETL ingestion into the cloud offers several benefits, primarily related to performance, cost, and efficiency. Cloud storage costs are typically based on the amount of data stored. Compressing data reduces its size, leading to lower storage costs. Most importantly, compressed data is smaller, which means it can be transferred more quickly over the network. This is particularly important when moving large volumes of data to the cloud, as it speeds up the ingestion process and reduces the load on network resources.

### Encrypt Data
Although not a direct benefit of compression, smaller data sizes can make encryption and decryption processes more efficient, enhancing data security during transfer and storage.

Please take a look at this [Page](https://nuneskris.github.io/publication/Collect-Data-Extraction) where I get into details on a couple of case studies. 

## Store

### Raw Layer
We need to compile raw data so that it can be picked up for subsequent processing. We call the data in this stage as raw and the layer in the architecture as raw. We define policies around access, archival, compliance and metadata management in this layer.

I have written best practices for this later in this [page](https://nuneskris.github.io/publication/DataStoreRawLayer).

### Lakehouse Storage Layer
There has been interesting movement in modern storae layer to enable reality of a lakehouse. I have a simple explaintion of what I call [Lakehouse (Modern) Storage Layer](https://nuneskris.github.io/publication/DataAnalytics-Storage-2024). There is basically three components of the storage layer in a data analytics architecture. Where are we going to store it? Cloud Storage; How is the data stored? FileFormat; How do we interact with the stored data? Table Format. It is very important to know your storage layer.

# Curate
Organizing, cleaning, and enriching the collected data to ensure its quality and usability. 
This process involves removing errors, filling gaps, and adding relevant metadata.

# Integrate
Combining curated data from different sources into a unified view. 
This involves aligning formats, schemas, and identifiers to ensure consistency and accessibility.

# Share
Distributing the integrated data to stakeholders, systems, and applications. 
This enables informed decision-making, collaboration, and operational efficiency.

# Organize
Data Organizing involves structuring and categorizing data in a systematic way to make it more accessible and useful. 
This includes creating logical and physical data models, defining data hierarchies, and implementing data taxonomies. 
The goal is to ensure that data is stored efficiently and can be easily retrieved and analyzed.

# Describe
Data Describing refers to the process of documenting data to provide context and meaning. 
This often involves creating metadata, which includes details like data definitions, data types, relationships, and business rules. 
Describing data helps users understand its structure, source, and how it should be used, enhancing data transparency and usability.

# Implement
Data Implementing encompasses the deployment of data solutions and systems. 
This involves setting up databases, data warehouses, data lakes, and other storage solutions, as well as implementing data processing workflows and pipelines. 
The implementation phase ensures that data infrastructure supports the organization’s data strategy and operational needs. 
