---
title: "Data Analytics Architectures"
excerpt: "A Brief History <br/><img src='/images/portfolio/DataArchitectures.png'>"
collection: portfolio
venue: 'Engineering'
tags:
  - Engineering
  - Management
---

We can understand where we are currently by understanding how we got here.

## Early Data Processing (1960s-1970s):Mainframes
I started worrking in the begining of 2005 and the team which sat next to me was the mainframe team. Data processing was primarily handled by large mainframe computers. These systems were designed for batch processing and performed straightforward reporting tasks within a single, integrated system.

## Data Warehousing (1980s-1990s): OLAP and OLTP
The rise of Online Analytical Processing (OLAP) and Online Transaction Processing (OLTP) enabled businesses to manage and analyze data more effectively. OLTP systems supported transactional workloads, while OLAP systems facilitated multidimensional data analysis. Each business function typically had its own OLTP application, which limited compute capacity and had locked-in data models unsuitable for complex, cross-functional reporting.

To overcome these limitations, separate OLAP applications were developed. These systems used ETL (Extract, Transform, Load) processes to consolidate and transform data from multiple sources into a format optimized for analytics. The first step in this process was the creation of a comprehensive data model that integrated data from various sources. This approach, known as the ***Inmon Architecture***, used a third normal form (3NF) for data modeling. The 3NF models were not optimized for complex queries. To address these challenges, data was broken down into subsystems called data marts, and dimensional modeling was adopted to enhance query performance. This approach focused on simplifying data structures to support analytical queries more efficiently. This approach, known as the ***Kimbal Architecture***

The historical development of data analytics architectures has evolved from the mainframe era focused on batch processing and straightforward reporting, to the advent of data warehousing and the separation of transactional and analytical workloads to handle more complex, multidimensional data analysis. The transition included the adoption of comprehensive data models, the introduction of ETL processes, and the use of data marts and dimensional modeling to improve performance and manageability.

I would like to note that the Kimbal based architecture lives till today and is still used to build data warehouses. I started by career in data in working on data warehouses and ETL.

## Big Data Era (2000s): Information and Data Revolution
Data volumes grew beyond the capacity of single storage units. Solutions like Teradata and Netezza emerged, but the internet companies introduced more scalable technologies. I have no experience on this technology though they were around. Google developed MapReduce for processing, and Hadoop was used for distributed storage. This technological foundations marked the beginning of the Big Data era. Data was stored in files and table formats, not confined to strict SQL standards, leading to the rise of NoSQL databases. However, managing and interacting with these systems was difficult due to data immutability, meaning files could not be updated or deleted easily.

# Real-Time Analytics and Cloud Computing (2010s) :

We were focusing on data processed in batches. However, internet companies had use cases to process data as soon as it was generated. This meant we could not wait for data to be collected, stored, and then processed for curation and analysis. We needed to do this in-memory and immediately as the data was collected, and that's exactly what streaming technologies like Apache Spark enabled.

While big data showed a lot of promise, organizations struggled to configure pools of storage and compute systems to deliver it. This included not knowing how much of these systems they would need to procure and set up. There were many more failures than successes. With the rise of faster internet connectivity, companies started offering computing systems consumed by organizations over the internet. This cloud computing kick-started the next generation of transformation, which companies embraced to reduce their dependence on managing scalable infrastructure.

# Modern Data Architectures (2020s):
Internet companies again came to the rescue by developing technologies that could manage files in data lakes just like how we handle tables in an SQL database, like a data warehouse. Combining data lakes and data warehouses, we call this architecture Data Lakehouses. Additionally, there is a better organization of data from a centralized system to collaborative decentralized ownership called Data Mesh and unifying data management end-to-end called Data Fabric.

These architectures reflect the ongoing evolution and adaptation to increasing data volumes, real-time processing needs, and the shift towards more flexible and scalable data management solutions.
