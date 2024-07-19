---
title: "Data Analytics Storage in 2024"
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

I have written a focused article with a [demo on Parquet](https://nuneskris.github.io/talks/Parquet-BestPracticeDemo) which describing its main features features. Below are the main featues.

* Parquet and ORC are preferred for analytical workloads due to their columnar storage format, which optimizes read performance and compression.
* Avro is preferred for write-heavy workloads and data interchange scenarios due to its row-based storage format and efficient serialization/deserialization.
* All three formats support schema evolution, making them flexible for changing data structures.
* Their wide compatibility with big data tools and platforms makes them popular choices in modern data engineering projects.

# Table Format
Managing data in files across distributed storage has evolved significantly to mirror the capabilities of a traditional data warehouse. A table format organizes data files and allows interaction with the data within these files as if they were a single table. It includes the schema representing the data within the files, a timeline/history of records updated or deleted, and supports efficient row-level ACID operations for updates and deletes, as well as data partitioning for query performance.

Hive was the technology of choice for early data lakes, but it had many limitations. Its primary shortcoming was its poor performance with interactive queries and lack of robust support for updates and deletes.

Data warehouses allow updates and transformations of data within the same system, a feature that was missing in early data lakes. With the introduction of table formats like Iceberg, Hudi, and Delta Lake, we can now merge the best features of both data lakes and data warehouses. This evolution is known as the Data Lakehouse architecture. Among these technologies, Iceberg is seeing significant adoption.

## Key Features Comparison: Lakehouse Formats vs. Data Lake Formats (Hive)
These Lakehouse table formats deliver significant advantages in terms of performance, flexibility, and ease of management over Apache Hive. They are more aligned with the needs of modern data engineering, providing robust support for both streaming and batch processing, schema and partition evolution, and efficient data management.

<img width="954" alt="image" src="https://github.com/user-attachments/assets/baddcf74-27fd-405a-9a75-ae3602a9300a">

### Schema Evolution and Data Management
Hive: Limited support for schema evolution, which often requires complex operations and can lead to bugs.
Lakehouse Formats (Iceberg, Hudi, Delta Lake): Robust support for schema evolution, allowing for adding, dropping, and renaming columns without needing to rewrite entire tables. They maintain a history of schema changes.
### ACID Transactions
Hive: Some support for transactional tables, but not as robust or efficient as modern table formats.
Lakehouse Formats: Provide reliable and consistent handling of data operations such as deletes and updates, supporting atomic operations.
### Performance and Scalability
Hive: Performance has been limited by reliance on the Hadoop MapReduce framework. Newer versions use Tez or Spark, but still lag behind modern table formats in terms of performance optimizations and scalability.
Lakehouse Formats: Designed for better performance and scalability, enabling support for interactive analytics.
### Efficient Upserts and Deletes
Hive: Handles upserts and deletes inefficiently, often requiring entire partitions to be rewritten.
Lakehouse Formats: Support efficient upserts (update or insert) and deletes, making them suitable for use cases like Change Data Capture (CDC). Hudi, in particular, is designed for streaming data ingestion and near real-time analytics.
### Data Versioning and Time Travel
Hive: Does not natively support time travel queries or data versioning.
Lakehouse Formats: Support time travel queries, allowing users to query historical versions of data easily. This is useful for debugging, auditing, and rollback scenarios.
### Partition Evolution
Lakehouse Formats: Supports partition evolution, which means you can change the partitioning scheme of a table without rewriting all the data. This is useful for optimizing query performance over time..
Hive: Partition management in Hive is static and changing the partitioning scheme usually requires significant data rewriting and operational overhead.
### Data Compaction and Optimization
Lakehouse Formats: Automatically handles compaction and data file optimization, reducing fragmentation and improving query performance.
Hive: Data compaction and optimization typically require manual intervention and custom tooling, which can be cumbersome and error-prone.
