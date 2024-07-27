---
title: "Data: Warehouse, Lake and Lakehouse"
excerpt: "Yesterday, Today and Tomorrow of Data Processing <br/><img src='/images/portfolio/DataArchitectures.png'>"
collection: portfolio
venue: 'Processing'
tags:
  - Engineering
  - Processing
---

I briefly referred to these three data architectures in the [page](https://nuneskris.github.io/portfolio/2-2-0DataAnalyticsArchitectures/) where discussed about the history of analytics data architectures.

We will spend some more time to discuss the comparisons of these three data architectures as we use them in 2024.

<img src='/images/portfolio/DataArchitectures.png'>

# Data Warehouse
A data warehouse is a ***centralized repository*** designed to store structured data from various sources within the system and the data model is optimized for query performance via SQL.Since we have to load data into a model which is predefined (schema-on-write) data needs to transformed and cleaned before it is loaded (ETL - Extract, Transform, Load). The underlyting architecture is based on RDMS system which allows consistency and integrity and there by enforces data consistency and integrity through ACID transactions. This allows strong data governance and quality. Business intelligence, reporting, and complex analytics are the main usecases of Data warehouses.

We have been building systems based on this architecture for more than 3 decades now and we know enough about it build integrated technologies, talent and development processes which ensures success. Snowflake brought life back into this architecture when organizations were either moving towards datalakes.

However they are very expensive to scale (Performance and data volume) and there was a high constraints on data neededing to be structured which required heavy ETL processes and development.

# Data Lake
There is a complete paradigm shift with datalakes. A data lake typically uses distributed storage like HDFS or Cloud storages. They can store vast amount of raw data in its native format (JSON, XML, CSV) including structured, semi-structured, and unstructured data. Rather than transform the raw data first and load data into a predefined schema, in data lakes we can ingest or laod data first in its raw form and then transform when we read. We refer to this as ELT - Extract, Load, Transform). We are able to  develop cost-effective storage solutions as we can use multiple commodity storage infrastructure for this and thus are highly scalable to accommodate large volumes of data. Big data analytics, machine learning, data exploration, and data science are the major use cases for this data architecture.

Can become a "data swamp" if not properly managed.
Slower query performance compared to data warehouses.
Requires robust data governance and metadata management.

# Data Lakehouse
A data lakehouse is a modern data architecture that combines the best features of data lakes and data warehouses. It provides the data management capabilities and high performance of a data warehouse with the flexibility and scalability of a data lake.

Key Characteristics:

* Unified Architecture: Combines the storage and processing capabilities of data lakes with the management and performance features of data warehouses.
* ACID Transactions: Ensures data consistency and reliability with ACID transactions.
* Schema Evolution: Supports schema changes without significant disruption.
* Flexible Storage: Stores data in various formats (e.g., Parquet, ORC) and supports structured, semi-structured, and unstructured data.
* Performance: Optimized for high-performance queries and analytics.
* Data Governance: Strong data governance and metadata management capabilities.
* Usage: Ideal for a wide range of analytics, including BI, data science, and real-time analytics.


Advantages:

Unified platform for all types of data and analytics.
Combines low-cost storage with high-performance analytics.
Supports real-time and batch processing.
Facilitates advanced analytics and machine learning.

Disadvantages:
Complexity in implementation and management.
Requires integration of various tools and technologies.



| Feature/Aspect          | Data Warehouse         |   Data Lake          |    Data Lake          |    
| ----------------------  | ---------------------- | -------------------- | --------------------- |
| Data Types              | Structured             | Structured, Semi-structured, Unstructured| All Types |
| Schema  | Schema-on-write        | Schema-on-read | Schema-on-read and write |
| Storage | Expensive, optimized for query speed | Cost-effective, scalable | Cost-effective, optimized |
| Performance | High | Variable | High |
| Data Processing | ETL (Extract, Transform, Load) | ELT (Extract, Load, Transform | ELT and ETL |
| Use Cases | BI, Reporting, Analytics | Big Data Analytics, Data Science | Unified Analytics, BI, Data Science |
| Data Consistency | Strong (ACID transactions) | Variable | Strong (ACID transactions) |
| Flexibility | Limited | High | High |
| Governance | Strong |Requires additional tools | Strong |

# Conclusion
Data Warehsourses Best for structured data and traditional business intelligence and reporting with high performance and strong data governance.
Data Lake: Ideal for storing vast amounts of raw data in various formats, supporting big data analytics, data exploration, and machine learning.
Data Lakehouse: Combines the strengths of both data warehouses and data lakes, providing a unified platform for diverse data types and analytics workloads, supporting advanced data management and high-performance queries.
