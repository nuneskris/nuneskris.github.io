---
title: "Data Analytics Architectures"
excerpt: "All can play their part<br/><img src='/images/portfolio/DataArchitectures.png'>"
collection: portfolio
venue: 'Engineering'
tags:
  - Engineering
  - Management
---

# A Brief History
We can understand where we are currently by understanding how we got here.

## Early Data Processing (1960s-1970s):Mainframes
I started worrking in the begining of 2005 and the team which sat next to me was the mainframe team. Data processing was primarily handled by large mainframe computers. These systems were designed for batch processing and performed straightforward reporting tasks within a single, integrated system.

## Data Warehousing (1980s-1990s): OLAP and OLTP
The rise of Online Analytical Processing (OLAP) and Online Transaction Processing (OLTP) enabled businesses to manage and analyze data more effectively. OLTP systems supported transactional workloads, while OLAP systems facilitated multidimensional data analysis. Each business function typically had its own OLTP application, which limited compute capacity and had locked-in data models unsuitable for complex, cross-functional reporting.

To overcome these limitations, separate OLAP applications were developed. These systems used ETL (Extract, Transform, Load) processes to consolidate and transform data from multiple sources into a format optimized for analytics. The first step in this process was the creation of a comprehensive data model that integrated data from various sources. This approach, known as the ***Inmon Architecture***, used a third normal form (3NF) for data modeling. The 3NF models were not optimized for complex queries. To address these challenges, data was broken down into subsystems called data marts, and dimensional modeling was adopted to enhance query performance. This approach focused on simplifying data structures to support analytical queries more efficiently. This approach, known as the ***Kimbal Architecture***

The historical development of data analytics architectures has evolved from the mainframe era focused on batch processing and straightforward reporting, to the advent of data warehousing and the separation of transactional and analytical workloads to handle more complex, multidimensional data analysis. The transition included the adoption of comprehensive data models, the introduction of ETL processes, and the use of data marts and dimensional modeling to improve performance and manageability.

I would like to note that the Kimbal based architecture lives till today and is still used to build data warehouses. 

## Big Data Era (2000s): Information and Data Revolution
Volume of data grew to volume which could not be containted with in a single storage unit. There were a few solutions like Terradata and Netezza (I have no clue working on it). The internet companies came to the rescue and build techologies leveraging commodity compute and storage by combining and corrdinating many of them to perform to scale and support the data volumes. Map-reduce by Google for data processing and Hadoop for storage was the foundation on which this architecture was based on we term this data as Big data.  Data was not stored using strict structured models and was stored as files and table formats which were not confined to strict SQL standards. These databases which handled unstructured and semi-structured data are called NoSQL databses.

# Real-Time Analytics (2010s):



In-Memory Computing: Technologies like Apache Spark enabled real-time data processing.
Cloud Computing: Cloud platforms offered scalable storage and computing resources.

Modern Data Architectures (2020s):
Data Lakes: Use of data lakes for storing raw, unprocessed data.
Data Lakehouses: Combining data lakes and data warehouses to unify analytics.
Data Mesh and Data Fabric: Emphasis on decentralized data ownership (Data Mesh) and unified data management frameworks (Data Fabric).
