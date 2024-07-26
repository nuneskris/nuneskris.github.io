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
We need to gather data from various sources, such as databases, applications, streams, or external sources in to the data platform where we want to process it. In this stage we need to
* Connect to the source systems and ingesting data into the data platform periodically (streaming mode or batch mode). We use terms such as ingest or extract for this.
* We need to compile raw data so that it can be picked up for subsequent processing.
* We call the data in this stage as raw and the layer in the architecture as raw. We define policies around access, archival, compliance and metadata management in this layer.
* I have written best practices for this later in this page.
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
