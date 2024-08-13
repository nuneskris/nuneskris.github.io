---
title: "Data Management"
excerpt: "Making Data Useful and Integrated: <br/><img src='/images/portfolio/DataManagement.png'>"
collection: portfolio
venue: 'Engineering'
tags:
  - Engineering
  - Management
---
<img width="1000" alt="image" src="/images/portfolio/DataManagement.png">

We want transform data into information which can lead to business insights by taking data from multiple business functions to make useful and integrated. 
Now we are getting into the realms where many architecture solutions come into play to realize this. Irrespective of the architecture solution, there are key management capabilities each of them try to solve. They are

# Data Processing Capabilities.

## [Collect](https://nuneskris.github.io/portfolio/2-1-1CollectArchitecture/)
Source systems are typically designed for transaction processing and cannot curate, transform, or integrate data within themselves. We need to collect data from various sources, such as databases, applications, streams, or external sources, into a data platform for processing. This data is usually collected into cloud storage, which is partitioned into a separate layer of the larger data analytics architecture. This layer is what we call the raw layer and data is stored in it the original source system state.

<img src='/images/portfolio/CollectArchitecture.png'>

The sub components to the collect architecture are below.

* Profile: Understand the source data's structure and consistency so we can design and plan the data for analytics processing.
* Capture: Define, isolate and filter only the data which is required for analytics processing.
* Extract: Move the data from source system into the target system enviorment. (Usually On-prem/Cloud to Cloud).
* Load: Loading the data into the analytics processing system incrementally.

I get into more details about collect architecture in in this [page](https://nuneskris.github.io/portfolio/2-1-1CollectArchitecture/).

## Store

### Raw Layer
Stroe data from collection: Gathering and aggregating data from various sources.

We need to compile raw data so that it can be picked up for subsequent processing. We call the data in this stage as raw and the layer in the architecture as raw. We define policies around access, archival, compliance and metadata management in this layer.

I have written best practices for this later in this [page](https://nuneskris.github.io/publication/DataStore-RawLayer).

### Lakehouse Storage Layer
There has been interesting movement in modern storae layer to enable reality of a lakehouse. I have a simple explaintion of what I call [Lakehouse (Modern) Storage Layer](https://nuneskris.github.io/publication/DataAnalytics-Storage-2024). There is basically three components of the storage layer in a data analytics architecture. Where are we going to store it? Cloud Storage; How is the data stored? FileFormat; How do we interact with the stored data? Table Format. It is very important to know your storage layer.

# Curate
Organizing, cleaning, and enriching the collected data to ensure its quality and usability. This process involves removing errors, filling gaps, and adding relevant metadata.

<img src='/images/portfolio/CurateProcess.png'>

## Data Cleansing
Data cleansing, also known as data scrubbing, refers specifically to the process of detecting and correcting (or removing) corrupt or inaccurate records from a dataset. The primary goal of data cleansing is to improve data quality by fixing errors and inconsistencies.

### Validation

## Error Handling
Fixing known errors in data (e.g., typos, incorrect values).

## Conform
Ensuring data follows a common format (e.g., date formats, text casing). Data Annotation Adding metadata, tags, or annotations to enhance understanding and usability.




# Integrate
Combining data from different sources into a unified dataset.

## Data Transformation
Changing the structure or format of data to make it more useful or compatible.

## Data Preservation
Ensuring long-term storage and accessibility of data.

## Data Governance
Establishing policies and procedures to manage data quality, security, and privacy.

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

# Related Pages
* https://nuneskris.github.io/portfolio/2-1-1CollectArchitecture/
