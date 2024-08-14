---
title: "Data Management"
excerpt: "Making Data Useful and Integrated: <br/><img src='/images/portfolio/DataManagement.png'>"
collection: portfolio
venue: 'Engineering'
tags:
  - Engineering
  - Management
---

# Table Contents
1. [Collect](#Collect)
2. [Store](#Store)
3. [Curate](#Curate)

<img width="1000" alt="image" src="/images/portfolio/DataManagement.png">

We want transform data into information which can lead to business insights by taking data from multiple business functions to make useful and integrated. 
Now we are getting into the realms where many architecture solutions come into play to realize this. Irrespective of the architecture solution, there are key management capabilities each of them try to solve.


<a name="Collect"></a>

# [Collect](https://nuneskris.github.io/portfolio/2-1-1CollectArchitecture/)
Source systems are typically designed for transaction processing and cannot curate, transform, or integrate data within themselves. We need to collect data from various sources, such as databases, applications, streams, or external sources, into a data platform for processing. This data is usually collected into cloud storage, which is partitioned into a separate layer of the larger data analytics architecture. This layer is what we call the raw layer and data is stored in it the original source system state.

<img src='/images/portfolio/CollectArchitecture.png'>

The sub components to the collect architecture are below.

* [Profile](https://nuneskris.github.io/publication/CollectDataProfiling): Understand the source data's structure and consistency so we can design and plan the data for analytics processing.
* [Capture](https://nuneskris.github.io/publication/Collect-Data-Capture): Define, isolate and filter only the data which is required for analytics processing.
* [Extract](https://nuneskris.github.io/publication/Collect-Process-Pre-ingest-vs-post-ingest): Move the data from source system into the target system enviorment. (Usually On-prem/Cloud to Cloud).
* [Load](https://nuneskris.github.io/publication/Collect-ExtractLoad-Patterns): Loading the data into the analytics processing system incrementally.

I get into more details about collect architecture in in this [page](https://nuneskris.github.io/portfolio/2-1-1CollectArchitecture/).

<a name="Store"></a>

# Store

<img src='/images/portfolio/StoreArchitecture.png'>

## [Raw Layer](https://nuneskris.github.io/publication/DataStore-RawLayer)

The raw layer is the initial landing zone or area where data is ingested directly from various source systems (e.g., databases, APIs, files, IoT devices). This layer holds the unprocessed, unfiltered data exactly as it was received from the source. We need to organize raw data so that it can be picked up for subsequent processing. We call the data in this stage as raw and the layer in the architecture as raw. We define policies around access, archival, compliance and metadata management in this layer. 

> 📝 Note: Essentially, Data is stored in its native format, without any transformations and it should be desifned to accomodate includes all multiple formats and type of data, often including redundant, irrelevant, or erroneous data. Data should be immutable since it is typically not altered or deleted once it is ingested, ensuring that the raw layer acts as a historical archive.

I have written best practices to design the Raw Layer in this [page](https://nuneskris.github.io/publication/DataStore-RawLayer).

### [Lakehouse Storage Layer](https://nuneskris.github.io/publication/DataAnalytics-Storage-2024)
There has been interesting movement in modern storae layer to enable reality of a lakehouse. I have a simple explaintion of what I call [Lakehouse (Modern) Storage Layer](https://nuneskris.github.io/publication/DataAnalytics-Storage-2024). There is basically three components of the storage layer in a data analytics architecture. Where are we going to store it? Cloud Storage; How is the data stored? FileFormat; How do we interact with the stored data? Table Format. It is very important to know your storage layer.

## Curated Layer

The data in the raw layer is stored in a file and format chosen for ease of extraction and loading into the analytics platform. We have established that the raw layer should be read-only to prevent data corruption and ensure that the data can be reprocessed in case of failures. By its nature, raw data is often not immediately usable for analytics, with quality being a primary concern. Therefore, we require an isolated environment where raw data can be copied and [curated](#Curate) to ensure it is trustworthy and ready for confident use by consumers.

The curation layer within the data analytics platform is where data is cleaned, validated, and enriched. This layer is designed to prepare the raw data for further processing, making it more consistent and reliable.

> 📝 Note: It’s important to note that the data in the curated layer remains isolated within the source data boundaries and is not yet integrated with data from other domains.

Unlike the raw layer, data in the curated layer must be managed for quality. Multiple transformations are applied to clean the data, removing duplicates, errors, and inconsistencies. We also standardize data formats, units of measure, and naming conventions. Finally, this layer must support tools that can validate the quality of the data.

## Integrated Layer

Once data is cleansed and validated in the curation layer, it needs to be integrated to fully realize its potential. The integration layer is where curated data from various sources is combined, aggregated, and transformed into a unified, coherent dataset. This layer supports complex data transformations and is crucial for creating integrated datasets that provide a comprehensive view of entities like customers, products, or transactions.

The integration layer must support tools that can bring together data from multiple sources—whether logically or physically—and run complex data manipulation queries to merge and harmonize the data.

> 📝 Note:  Since the integrated data will have multiple consumers, this layer must be highly governed to ensure security, accessibility, and quality. However, typical consumers will extract the data as-is, without modifying it or running complex analytics queries, which are usually performed in the serving layer.

## Serving Layer


<a name="Curate"></a>

# [Curate](https://nuneskris.github.io/portfolio/2-1-2CurateArchitectrure/)

Data from source systems have multiple quality issues. One of the most important objectives of this stage is to improve the quality of the data so it is usable. I explain a simple process to achieve this below and explained in this [page](https://nuneskris.github.io/portfolio/2-1-2CurateArchitectrure/).

<img src='/images/portfolio/CurateProcess.png'>

* ***[Data Cleansing](https://nuneskris.github.io/publication/CurateDataCleansing)***: Data cleansing, also known as data scrubbing, refers specifically to the process of detecting and correcting (or removing) corrupt or inaccurate records from a dataset. The primary goal of data cleansing is to improve data quality by fixing errors and inconsistencies.
* ***Validation***: We also would need a process that ensures data is accurate and consistent to ensure it is usable so that it can be trusted by any consumers of the data.
* ***Error Handling***: Fixing known errors in data (e.g., typos, incorrect values).
* ***MetaData***:


# Integrate

> Coming Soon

# Share

> Coming Soon

# Organize

> Coming Soon

# Describe

> Coming Soon

# Implement

> Coming Soon

# Related Pages
* https://nuneskris.github.io/portfolio/2-1-1CollectArchitecture/
