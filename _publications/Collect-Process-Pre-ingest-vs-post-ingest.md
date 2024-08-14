---
title: "Collect: Pre-ingest vs Post-ingest Processing"
collection: publications
permalink: /publication/Collect-Process-Pre-ingest-vs-post-ingest
excerpt: 'I do not tend to draw hard lines between applying processing logic directly on the source system before extracting the data or performing transformations post-ingestion in an analytics platform. Both approaches are valid, depending on factors such as data volume, complexity, real-time requirements, and system architecture. However, most modern data scale needs require processing to be done post-ingestion.'
date: 2024-05-01
venue: 'Processing'
tags:
  - Collect
---

I do not tend to draw hard lines between applying processing logic directly on the source system before extracting the data or performing transformations post-ingestion in an analytics platform. Both approaches are valid, depending on factors such as data volume, complexity, real-time requirements, and system architecture. However, most modern data scale needs require processing to be done post-ingestion.

# Pre-ingest processing
Pre-ingestion transformation should be done only when these transformations are light and do not impact the source systems. Intensive transformations can put a strain on the source system, potentially affecting its primary operations. Complex business logic might be harder to manage directly on the source database, especially if multiple joins and aggregations are required. Once the data is transformed and ingested, it's more difficult to change the logic or perform different analyses without re-extracting the data.

I typically encourage this approach when: 
* Golden Rule of Thumb: When dealing with small to moderate data volumes where the source system can handle the load without significant performance degradation.
* Complex Transaction Handling: The number of tables can get bloated, especially with many lookup (reference) tables. Additionally, I've seen source tables hyper-normalize data to handle complex transactions. If the grain of these transactions complicates ETL pipelines, both in understanding them or processing them, it’s best to apply transformation before extraction. This is especially true when the source application team is best suited to handle the processing logic. This ensures data consistency by applying transformations and business logic once at the source, avoiding discrepancies that might arise from multiple transformations downstream.
* Urgent Use Cases: When data needs to get to reporting systems quickly for urgent use cases. These are typically throwaway work, but if the effort is small and we need to get data in front of analysts quickly, this approach can be allowed.

# Post-ingest processing
Minimizing load on the source system by extracting raw data ensures it can focus on its primary operations. We need to allow analysts to apply different transformation logics and experiment with data without impacting the source system or requiring re-extraction. Modern analytics platforms are user-friendly and optimized for varied data processing and can handle complex transformations efficiently. In most cases, this is the preferred way to process data.

### Collect is only to collect
> ***Case Study***: An organization's source applications team had more influence and secured a collection budget. They set up a team to extract data from the source system and convinced the business that they would provide near-ready data for analytics. The team developed complex extraction queries that required specialized skills tied to the source application. Five years later, when enhancements were needed, they couldn't find skilled resources within the budget to build upon the existing extraction scripts. I was called in to develop the components within budget and time. I quickly realized the effort required to continue with the current design. I utilized a single mid-experienced SQL developer and two experienced cloud analytics developers to completely migrate the entire solution. We simplified the extraction process in the source application and handled all the transformations in the analytics platform. We were able to deliver this within budget and ahead of schedule by 10%. This approach also reduced the effort needed for future enhancements by 70%.

# Understand the Source Data
Teams that work with the source applications know the source data best. Every hop data takes away from the source, the knowledge of the source data degrades. In my opinion, most delays in development teams occur because they struggle with the source model. They have decent clarity on the target models but require constant support from source application data SMEs, which creates bottlenecks and rework.
> ***Case Study***: The best option is to include a member from the source team within the ETL team. This happened only once in my experience, but the resource was able to incredibly improve communication efficiencies and query resolution, fast-tracking the development effort. The resource may not have known ETL and cloud data engineering, but they knew the source data model and the data intimately.

# Documentation is key here
We would need to leverage what is existing and build with new know uncovered. Discussions between the source application/data team and the ETL happen on email and informal discussions. Formally record these discussions before they are lost.

# Some SQL Best Practices Applicable for Collect
1. Extract data which is needed. This help in not having to deal with columns and tables which are would not be consumed downstream.
2. Reduce Joins unless we want to avoid techical columns references.
3. Write Modular Queries: Writing modular queries for data extraction involves breaking down complex queries into smaller, reusable components. This approach enhances readability, maintainability, and reusability of SQL code. Use CTE, views, stored procedures to create modular queries.
4. Optimize Query Performance by leveraging existing indexes. Create new indexes if necessary to improve performance. Use appropriate join types and ensure joins are based on indexed columns. Apply filters early in the query to limit the dataset size using the WHERE clause.
5. Explicitly specify the columns you need rather than using SELECT * to reduce the amount of data transferred and processed.
6. Minimize the number of joins and avoid complex joins where possible.

# Develop to Handle Errors: 
1. Always develop to handle errors with detialed logging.
2. Requirements need to detailed for every type of possible error, with test cases.
3. Implement robust error handling to capture and manage extraction errors. Ensure the process can recover gracefully and test these scenarios.
4. Log extraction activities, including timestamps, row counts, and any errors encountered. This helps with monitoring and debugging.
5. Continously compare extracted data against source data to ensure accuracy and completeness.

# [Compresss data](https://nuneskris.github.io/publication/Collect-Data-Extraction-Compress)
Compressing data during ETL ingestion into the cloud offers several benefits, primarily related to performance, cost, and efficiency.

# [Batch Transfer Data](https://nuneskris.github.io/publication/Collect-Data-Extraction-Batch)
The most common scenario we would encouter is to move data in batches. We need to extract data from source applications using simple queries for each table, outputting the results into flat files.
