---
title: "Collect: Best Practices"
collection: publications
permalink: /publication/DataCollect-BestPractices
excerpt: ''
date: 2024-05-01
venue: 'Processing'
tags:
  - Collect
---

# Pre-ingest vs post-ingest
I do not tend to draw hard lines between applying processing logic directly on the source system before extracting the data or performing transformations post-ingestion in an analytics platform. Both approaches are valid, depending on factors such as data volume, complexity, real-time requirements, and system architecture. However, most modern data scale needs require processing to be done post-ingestion.

## Pre-ingest processing
Pre-ingestion transformation should be done only when these transformations are light and do not impact the source systems. Intensive transformations can put a strain on the source system, potentially affecting its primary operations. Complex business logic might be harder to manage directly on the source database, especially if multiple joins and aggregations are required. Once the data is transformed and ingested, it's more difficult to change the logic or perform different analyses without re-extracting the data.

I typically encourage this approach when: 
* Golden Rule of Thumb: When dealing with small to moderate data volumes where the source system can handle the load without significant performance degradation.
* Complex Transaction Handling: The number of tables can get bloated, especially with many lookup (reference) tables. Additionally, I've seen source tables hyper-normalize data to handle complex transactions. If the grain of these transactions complicates ETL pipelines, both in understanding them or processing them, it’s best to apply transformation before extraction. This is especially true when the source application team is best suited to handle the processing logic. This ensures data consistency by applying transformations and business logic once at the source, avoiding discrepancies that might arise from multiple transformations downstream.
* Urgent Use Cases: When data needs to get to reporting systems quickly for urgent use cases. These are typically throwaway work, but if the effort is small and we need to get data in front of analysts quickly, this approach can be allowed.

## Post-ingest processing
Minimizing load on the source system by extracting raw data ensures it can focus on its primary operations. We need to allow analysts to apply different transformation logics and experiment with data without impacting the source system or requiring re-extraction. Modern analytics platforms are user-friendly and optimized for varied data processing and can handle complex transformations efficiently. In most cases, this is the preferred way to process data.

### Collect is only to collect
> ***Case Study***: An organization's source applications team had more influence and secured a collection budget. They set up a team to extract data from the source system and convinced the business that they would provide near-ready data for analytics. The team developed complex extraction queries that required specialized skills tied to the source application. Five years later, when enhancements were needed, they couldn't find skilled resources within the budget to build upon the existing extraction scripts. I was called in to develop the components within budget and time. I quickly realized the effort required to continue with the current design. I utilized a single mid-experienced SQL developer and two experienced cloud analytics developers to completely migrate the entire solution. We simplified the extraction process in the source application and handled all the transformations in the analytics platform. We were able to deliver this within budget and ahead of schedule by 10%. This approach also reduced the effort needed for future enhancements by 70%.

# Understand the Source Data
Teams that work with the source applications know the source data best. Every hop data takes away from the source, the knowledge of the source data degrades. In my opinion, most delays in development teams occur because they struggle with the source model. They have decent clarity on the target models but require constant support from source application data SMEs, which creates bottlenecks and rework.
> ***Case Study***: The best option is to include a member from the source team within the ETL team. This happened only once in my experience, but the resource was able to incredibly improve communication efficiencies and query resolution, fast-tracking the development effort. The resource may not have known ETL and cloud data engineering, but they knew the source data model and the data intimately.

Documentation is key here. We would need to leverage what is existing and build with new know uncovered. Discussions between the source application/data team and the ETL happen on email and informal discussions. Formally record these discussions before they are lost.



# Write Modular Queries
Writing modular queries for data extraction involves breaking down complex queries into smaller, reusable components. This approach enhances readability, maintainability, and reusability of SQL code. Here are some best practices and strategies for writing modular queries:

# Some SQL Best Practices
1. Extract data which is needed. This help in not having to deal with columns and tables which are would not be consumed downstream.
2. Reduce Joins unless we want to avoid techical columns references.
3. 


1. Design time improvements
    1. End to end data engineering teams
    2. Extract necessary data only: The entire pipeline gets stressed when we extract more data than needed. 
    3. Secure data both at rest and transfer
    4. Ensure data quality at source
2. Build time improvements
    1. Breaking larger jobs into smaller ones
    2. Compress data whenever possible (Transfer and Archival)
    3. Manage error and reject record handling: Review if we can handle rejected records separately rather than failing the entire job.
    4. Review portioning of data for parallel or bulk processing
        1. A Salesforce Load component which took 7 hours was designed with a realtime load vs bulk load which improved the load time by 10 fold.
3. Runtime improvements
    1. Automate manual interventions across the pipeline
        1. There was a 4 hour job which had 8 stages under a single AutoSys sequence. These jobs would fail individually this forced the support engineers to keep monitoring each job fix an issue and trigger the subsequent jobs. Simple solution of having separate Austosys routine for each data stage job which is triggered from the end of the subsequent job, with delay alarms. 
    2. Appropriate logging, monitoring and alerting
    3. Strive to move maintenance tasks to L1. Ex: Files not arriving, Restart jobs etc.
