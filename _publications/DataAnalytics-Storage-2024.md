---
title: "Storage in Data Analytics Storage in 2024"
collection: publications
permalink: /publication/DataAnalytics-Storage-2024
excerpt: 'Storage is foundational. The choices are simplified by the maturing technolgies. Providing a technical overview'
date: 2024-05-01
venue: 'Storage'
slidesurl: ''
paperurl: 'http://academicpages.github.io/files/paper1.pdf'
---

There is basically three components of the storage layer in a data analytics architecture. Where are we going to store it? Cloud Storage; How is the data stored? FileFormat; How do we interact with the stored data? Table Format. It is very important to know your storage layer.

<img width="612" alt="image" src="https://github.com/user-attachments/assets/9675c2a4-8d5a-4c12-b9f7-bd8c46e42f13">

# Storage

## Best Practices for Building a Cloud Storage Layer
The choices for cloud storage solutions are straightforward, but deciding why to choose one over the other can be complex. The good news is that all three solutions can be chosen based on the preferences of each domain, and cleansed data can be moved around as needed. While this approach incurs costs, which we will discuss in another topic, the following best practices are applicable regardless of the cloud vendors (or a mix thereof) chosen.

### Decentralization and Data Control
As data increasingly moves towards decentralization, we may have solved the problem of data swamps but need to be vigilant about maintaining control over how data is stored within domains. I have written an article on [data domains](https://nuneskris.github.io/publication/Domain-Oriented-Business-Capability-Map) that provides guidance on the expectations for how domains should store data.

### Manage Buckets
Organizing data into storage buckets across data domains can be confusing, especially when maintaining data across different stages of the transformation process and categorizing data based on access. I have already written generic [best practices managing buckets](https://nuneskris.github.io/talks/CloudStorage-Best-Practices). 

### Data Volume
Determining how much data is needed for data analytics systems, regardless of the architecture involved (Data Lake, CDW, DLH, Data Mesh, Data Fabric, etc.), is challenging. Common questions include whether to include replicated data across layers, data copies, archived data, etc. The best approach is to measure data volume at the source applications. Use multiple metrics to gauge this, such as:

* Size (GB, TB)
* Number of tables
* Average number of rows and columns
* Data growth

This approach provides a good estimate of the data volume within an application, domain, and enterprise. Cost is a function of data size. Once we have a good idea on the size we are dealing with, we can apply cost management and monitoring on storage costs. The Cloud Vendors provide excellent management tools to track spending.

Utilize tiered storage solutions to balance performance and cost. Store frequently accessed data in high-performance storage and move infrequently accessed data to lower-cost storage tiers.

Define and enforce data retention policies to manage the lifecycle of data. Automatically archive or delete data that is no longer needed to reduce storage costs and minimize risk.

### Keep the Raw Layer "Raw"
The main objective is to ingest data into the raw layer quickly and efficiently, maintaining raw data in its original format. Do not apply any transformtion on the data. Implement data archival strategies for long-term storage of historical data. Use cost-effective storage solutions for archival data that is infrequently accessed but needs to be retained for compliance or historical analysis. Implement robust backup strategies to ensure data can be recovered in the event of data loss or corruption from the raw layer. Regularly test backup and recovery procedures.

#### Maintain Raw Data in Its Original Format
When extracting data from source applications, it's crucial to keep the data and file formats in their raw state. This approach ensures that the data is easily extractable and stays true to its source. Source application systems (tools, people, processes) are often not designed to manipulate data once it leaves the system. Instead of forcing these systems to process data into a prescribed format, focus on extracting the data efficiently in a manner that aligns with the source application's natural capabilities and limitations. Allow the source system to define the file sizes based on its constraints and let the processing occur in subsequent stages.

For example, in two separate organizations, data extraction was performed via REST APIs. This method proved to be slow and taxing on the source application, leading to delays and recovery difficulties. The data teams had standardized JSON as the data format for extraction, which may be appropriate for higher layers of the architecture but not ideal for initial extraction. The application teams were more than willing to use their existing REST APIs, but by directly querying the database for data extraction, performance improved by a factor of 20.

#### No Transformation or Changes to Raw Data
Data archives are maintained at this layer, serving as a rollback point for any processing. It's important to restrict end-user access to this layer. Use automatic policies to compress and archive data to reduce costs. No overriding is allowed, which means handling duplicates and different versions of the same data.

#### Organizing the Raw Layer
Within each domain, segregate the data based on the source system. Further partitioning based on the time of ingestion helps with archiving and retrieval. Multiple tags can be used on the ingested objects, such as the time of batch ingestion at the source, which is used to calculate cutoff times.

### Security and Access Management
Data Encryption: Ensure that all data, both at rest and in transit, is encrypted using industry-standard encryption methods. This helps protect sensitive information and comply with regulatory requirements. Access Controls: Implement strict access controls using IAM (Identity and Access Management) policies to ensure that only authorized users and services have access to the data. Regularly review and update these policies to maintain security. Enable logging and monitoring to track data access, usage patterns, and system performance. Use these logs to detect anomalies, troubleshoot issues, and optimize performance. Regularly audit data access  and usage to to production areas to ensure compliance with internal policies and regulatory requirements. Maintain detailed audit logs for accountability and transparency.

# File Format

# Table Format

There has been large efforts to manage data in files across distributed storage like we do in a datwawarehouse.  A table format oraganizes data files and interacts with the data within the files as a single table. It contains the schema of the table representing the data within the files,timeline/history of records updated or deleted into the files, data within the files to support efficient  row-level ACID operation for updates/deletes, and data-partitions for query performance.  

Hive was the technology of choice and a large part of how we developed data lakes but they had many limitations. According to me, the main limitation was its shortcomming with interactive queries due to it slow performance and it not support update and deletes.

The key to datawarehouses was we could update the data with new records and transform the data for serving within the same system. With the introduction of Table formats Iceberg, Hudi and Delta Lake, we are able to merge the best of both Data Lakes and Datawarehouse. This is the Data Lakehouse architecture. I believe these 3 technologies will provide the same features and the choice would not make a difference. I am seeing larger adoption of Iceberg.

## The key features comparison between Lakehouse formats and Datalake formats (Hive)

***Schema Evolution and Data Management***: Schema evoles all the time. We race to deliver MVPs so that we can get buy-in and incremently add featurs with agile development. Hive has limited support for schema evolution which requires mightmare operations which leads to multiple bugs.  The mordern tables provide robust support for schema evolution, allowing for adding, dropping, and renaming columns without needing to rewrite entire tables. It maintains a history of schema changes.

***ACID Transactions***: While Hive does have some support for transactional tables, it is not as robust or efficient as the support found in Iceberg, Hudi and Delta Lake. This means that the analytics system can reliably and consistently handle data operations such as delete and update atomically.

***Performance and Scalability***: Traditionally, Hive's performance has been limited by its reliance on the Hadoop MapReduce framework. Although newer versions of Hive use Tez or Spark as execution engines, it still lags behind Iceberg and Hudi in terms of performance optimizations and scalability. This enables interactive analytics support.

***Efficient Upserts and Deletes***: Hive handles upserts and deletes in a cumbersome and inefficient way, often requiring workarounds such as rewriting entire partitions. Iceberg supports upserts (update or insert) and deletes efficiently, making it suitable for use cases like Change Data Capture (CDC). Hudi specializes in handling upserts and deletes, designed for streaming data ingestion and near real-time analytics.

***Data Versioning and Time Travel*** Hive: Does not natively support time travel queries or data versioning, making it less flexible for historical data analysis. The Lakehouse table formats supports time travel queries, allowing users to query historical versions of data easily. This is useful for debugging, auditing, and rollback scenarios.
