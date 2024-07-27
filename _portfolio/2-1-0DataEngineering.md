---
title: "Making Data Useful and Integrated: Data management"
excerpt: " <br/><img src='/images/portfolio/DataManagement.png'>"
collection: portfolio
venue: 'Engineering'
tags:
  - Engineering
  - Management
---
<img width="612" alt="image" src="/images/portfolio/DataManagement.png">

We want transform data into information which can lead to business insights by taking data from multiple business functions to make useful and integrated. 
Now we are getting into the realms where many architecture solutions come into play to realize this. Irrespective of the architecture solution, there are key management capabilities each of them try to solve. They are

Data Processing Capabilities.

# Collect
Source systems are typically designed for transaction processing and cannot curate, transform, or integrate data within themselves. We need to collect data from various sources, such as databases, applications, streams, or external sources, into a data platform for processing. This data is usually collected into cloud storage, which is partitioned into a separate layer of the larger data analytics architecture. This layer is what we call the raw layer and data is stored in it the original source system state.

## Data Profiling
We need to analyze the source data to understand its structure and consistency. I strongly believe, and frequently emphasize to the teams I lead, that data engineering does not deal with large problems but rather a large number of small problems related to data issues. The main objective of profiling data is to scope the size of the data engineering effort, which is a function of the complexity of the structure and inconsistencies of the data.
> Very often, we develop data engineering pipelines based on test data that does not reveal the true extent of the data quality problems we will encounter in production scenarios. This is why it is important to break down data engineering projects into smaller, end-to-end agile cycles where we test the pipelines with production data early, rather than face inevitable surprises from bad data.
> Invest in data profiling tools. I have seen organizations trying to write numerous queries to understand the data. Data engineering is about extracting and transforming data, not writing throwaway queries to understand it.

## Data Capture
Data residing in the source application needs to be captured for an initial load when moving into production, along with updates to the data (deletions, edits, and insertions) during each periodic transfer. These updates are referred to as change data capture, which is the most critical task in the collect component.
> In some cases, even the initial load may be too large for the source system to handle, requiring it to be broken into smaller batches. These batches need to be tested early. Additionally, we need the capability to rebase the entire dataset from the source when issues with the data arise.

## Data Extraction
Typically we would capture the data changes as files. Connect to the source systems and ingesting data into the data platform periodically (streaming mode or batch mode). We use terms such as ingest or extract for this. There are essentially only one wa extracting data espcially from on-prem to cloud. Compress and batch transfer as encrypt in flight. 

## Raw Layer
* We need to compile raw data so that it can be picked up for subsequent processing. We call the data in this stage as raw and the layer in the architecture as raw. We define policies around access, archival, compliance and metadata management in this layer.

I have written best practices for this later in this [page](https://nuneskris.github.io/publication/CollectDataArchitecture)

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
