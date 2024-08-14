---
title: "Storage Architecture"
excerpt: "Storage for Data Analytics: Coming Soon <br/><img src='/images/portfolio/StoreArchitecture.png'>"
collection: portfolio
venue: 'Data Management'
tags:
  - Curate
---

<img width="920" alt="image" src="/images/portfolio/StoreArchitecture.png">

In a data architecture, different layers are used to organize and process data as it moves from its raw form to its final, consumable state. Each layer has a specific role in the data pipeline, facilitating the transformation, enrichment, and preparation of data for analysis and reporting. 

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


