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

# Storage
The choices are simple and the decision as to why choose one over the other is complex. The good news is we can choose all three based on preferences of each domains and move cleansed data around. There is a cost to this but we will get in this in another topic. However, irrespective of the cloud vendors (or a mix) which are chosen, below are the best practices into how we build the storage layer.

More and more data is moving towards decentralization. By this, we may have solved a data swamp, but we need to be careful in loozing control on how data is stored within domains. I have written an article on my thouhts on [data domains](https://nuneskris.github.io/publication/Domain-Oriented-Business-Capability-Map). There needs to be clear guidance on the expectations on how domains store data.

## Manage Buckets
There is often confusion in oraganing data into storgae buckets across data domains, maintaining data across different stages as it goes through the transformation process and categorization of data based access. I have already written generic [best practices managing buckets](https://nuneskris.github.io/talks/CloudStorage-Best-Practices). 

### Data Volume
How much data is there to store data analytics sytems irrespective of the arhictecture involved (Data Lake,CDW, DLH, DataMesh, Data Fabric etc)? To answer this questions is tough. Very often we get quesions like, do we need to include replicated data across layers, data copies, archived data etc. The way I like to measure this is, going to the source applications and measure volume there. Use multiple ways to measure such as (1) Size (GB, TB), (2) number of tables, (3) average number of rows and columns (4) data growth. This gives a good idea on how much of data we are dealing with within a application, domain and enterprise.

### Keep Raw Layer "raw"
The main objective is to ingest data into Raw as quickly and efficiently.
### Maintain raw data in its original format.
When extracting data from source applications, it's crucial to maintain the data and file formats in their raw state. This approach ensures that the data is easily extractable and stays true to its source. Source application systems (tools, people, processes) are often not designed to manipulate data once it leaves the system. Instead of forcing these systems to process data into a prescribed format, focus on extracting the data efficiently in a manner that aligns with the source application's natural capabilities and limitations. Also let the source system define the size of the files based on its constraints. Let the processing happen in the subsequent stages.

For example, in two separate organizations, data extraction was performed via REST APIs. This method proved to be slow and taxing on the source application, leading to delays and recovery difficulties. The data teams had standardized JSON as the data format for extraction, which may be appropriate for higher layers of the architecture but not ideal for initial extraction. The application teams were more than willing to use their existing REST APIs, but by directly querying the database for data extraction, performance improved by a factor of 20.

### No Transformation or changes to raw data
Data archive is maintained at this layer we use this as a point to rollback from an processing. Hense it is important to restrict access to end users to this layer. Use automatic policies to compress and arhive data to reduce cost.
No overriding is allowed, which means handling duplicates and different versions of the same data. 

### Organizing raw layer
Within each domain, we segregate the data based on the souce system. Further partition based on time of ingestion helps in archival and retrival. There are multiple tags which can used on the objects which are ingested for example the time of batch ingestion at source (used to calcualte cutoff).

# Table Format

There has been large efforts to manage data in files across distributed storage like we do in a datwawarehouse.  A table format oraganizes data files and interacts with the data within the files as a single table. It contains the schema of the table representing the data within the files,timeline/history of records updated or deleted into the files, data within the files to support efficient  row-level ACID operation for updates/deletes, and data-partitions for query performance.  

Hive was the technology of choice and a large part of how we developed data lakes but they had many limitations. According to me, the main limitation was its shortcomming with interactive queries due to it slow performance and it not support update and deletes.

The key to datawarehouses was we could update the data with new records and transform the data for serving within the same system. With the introduction of Table formats Iceberg, Hudi and Delta Lake, we are able to merge the best of both Data Lakes and Datawarehouse. This is the Data Lakehouse architecture. I believe these 3 technologies will provide the same features and the choice would not make a difference. I am seeing larger adoption of Iceberg.

The key features comparison.
### Schema Evolution and Data Management
Schema evoles all the time. We race to deliver MVPs so that we can get buy-in and incremently add featurs with agile development. Hive has limited support for schema evolution which requires mightmare operations which leads to multiple bugs.  The mordern tables provide robust support for schema evolution, allowing for adding, dropping, and renaming columns without needing to rewrite entire tables. It maintains a history of schema changes.

### ACID Transactions
Iceberg: Supports ACID (Atomicity, Consistency, Isolation, Durability) transactions, enabling reliable and consistent data operations.
Hudi: Also supports ACID transactions, ensuring that operations like updates and deletes are handled atomically.
Hive: Does not natively support ACID transactions. While Hive does have some support for transactional tables, it is not as robust or efficient as the support found in Iceberg or Hudi.
3. Performance and Scalability
Iceberg: Designed for high performance and scalability, Iceberg can handle petabyte-scale datasets efficiently. It optimizes data layout and indexing, improving query performance.
Hudi: Optimizes data layout and supports indexing, which improves read and write performance, especially for large-scale incremental data processing.
Hive: Traditionally, Hive's performance has been limited by its reliance on the Hadoop MapReduce framework. Although newer versions of Hive use Tez or Spark as execution engines, it still lags behind Iceberg and Hudi in terms of performance optimizations and scalability.
4. Efficient Upserts and Deletes
Iceberg: Supports upserts (update or insert) and deletes efficiently, making it suitable for use cases like Change Data Capture (CDC).
Hudi: Specializes in handling upserts and deletes, designed for streaming data ingestion and near real-time analytics.
Hive: Handling upserts and deletes in Hive is cumbersome and inefficient, often requiring workarounds such as rewriting entire partitions.
5. Data Versioning and Time Travel
Iceberg: Supports time travel queries, allowing users to query historical versions of data easily. This is useful for debugging, auditing, and rollback scenarios.
Hudi: Also supports data versioning and time travel, enabling users to query data as it was at a specific point in time.
Hive: Does not natively support time travel queries or data versioning, making it less flexible for historical data analysis.
6. Compatibility and Ecosystem Integration
Iceberg: Integrates well with modern data processing frameworks like Apache Spark, Flink, and Presto. It also has broad compatibility with various data formats such as Parquet, Avro, and ORC.
Hudi: Designed to work seamlessly with Apache Spark and integrates with other tools in the Hadoop ecosystem. It supports popular data formats like Parquet.
Hive: While Hive integrates well with the Hadoop ecosystem, its compatibility with modern data processing frameworks and newer data formats is more limited compared to Iceberg and Hudi.
7. Metadata Management
Iceberg: Manages metadata efficiently, allowing for fast table operations and reducing the overhead associated with large tables.
Hudi: Also focuses on efficient metadata management, optimizing for scenarios with frequent data changes.
Hive: Metadata management in Hive can become a bottleneck, especially for large datasets with many partitions.

