---
title: "Collect - Extraction: Batch Transfer"
collection: publications
permalink: /publication/Collect-Data-Extraction-Batch
excerpt: 'Batch Transfer: One of the most common scenarios of extraction. For now atleast'
venue: 'Processing'
date: 2024-05-01
tags:
  - Collect

We would need to extract data from source applications which is accmplished by using a simple query for each table into flat files which then need to be ingested into analytics platform which are in the cloud. I have seen way too often pipelines makes the biggest mistakes in this subsystem where we move data from on-prem to cloud. 
This is usually because we have 2 teams who would need to coordinate there. 
The application team who owns the source application and the data teams for owns the analytics applications. 
I would like to provide a framework to build a subsystem architecture which would provide to ensure efficiency, reliability, and security.

# Extraction Query
Extraction queries need to be free from failures. Mordern business applications provide a decent data integrity checks before persisting them into their systems.
A good rule of thumb is to not to have too much of pre-ingest processing and have straight forward queries to extract the data as a batch and relying only on time windows to filter.
However if there is a need to apply processing logic on the extract query, we would need to ensure these queries dont lead to failures.

However I have seen many failures here due to
1. Syntax Errors from Schema Evolution: Table schema changes without informing pipeline teams and leads to disruptions. This is a larger change and governanc problem which will not be discussed here, but we would need to embrace this and design our queries to handle it.
2. Query Memory and CPU Resource Limitations: Many source applications have this limit. So breaking queries down into sub-batches is recommended.
3. Disk Space where we extract files: This typically happens in extracting very large data sets causing performance degradation or failures. I have seen a P1 incident becaus of this. Insufficient disk space for temporary files or output. I recommend deleting the files from on-prem once they are transfered and using the cloud to archive.
4. Avoid Query Complexity:  Complex joins, subqueries, heavy aggregation and grouping operations can lead to performance issues or timeouts.
5. Coordinate System Maintenance downtimes: Unexpected maintenance windows during extraction schedules fails extraction queries. Another maintenance issue I have seen.
6. Permission and Access Issues of the query. Ensure we have stable access permissions here. Use service agent and not named acess. This happens even today because ncessary permissions to access data or execute queries can change or acces previously granted but later revoked.
7. Data Issues
Data Corruption: Corrupted data files or records.
Inconsistent Data Types: Mismatched data types causing casting errors.
Null Values: Unexpected null values causing logic errors.
8. Concurrency Issues
Locking and Deadlocks: Simultaneous queries causing locks or deadlocks.
Resource Contention: Multiple queries contending for the same resources.
9. Network Issues
Connectivity: Network interruptions or latency affecting data source connectivity.
Timeouts: Network timeouts due to slow responses or heavy traffic.

11. Configuration and Environment
Configuration Errors: Misconfigurations in database settings or query options.
Environment Changes: Changes in the execution environment, such as database upgrades or migrations.
12. External Dependencies
Service Dependencies: Dependencies on external services or APIs that might be down or slow.
File Dependencies: Missing or inaccessible external files or data sources.
13. Execution Plan Changes
Optimizer Plans: Changes in the database optimizer's execution plan leading to suboptimal performance.
Statistics: Outdated or inaccurate statistics affecting query planning.

17. Data Source Changes
Schema Changes: Changes in the data source schema leading to compatibility issues.
Data Source Moves: Migration of data sources to new locations without updating the query configuration.





ee
-----


----

Data Cleaning: Ensure that the data is clean and free of errors before the transfer. This reduces the risk of corrupt data causing issues in the target system.
Data Validation: Validate the data to ensure it meets the expected format and schema requirements.
2. Efficient Data Transfer
Compression: Compress data before transfer to reduce the amount of data being sent over the network, which can speed up the transfer and reduce costs.
Chunking: Break down large datasets into smaller, manageable chunks to avoid overwhelming the network and to facilitate easier retries in case of failure.
3. Security
Encryption: Encrypt data in transit using secure protocols (e.g., TLS) to protect sensitive information from being intercepted.
Authentication: Use strong authentication methods to ensure that only authorized users and systems can initiate data transfers.
Access Controls: Implement strict access controls to limit who can access the data and the transfer mechanisms.
4. Reliability and Monitoring
Retry Mechanism: Implement retry logic to handle transient network issues and ensure that data transfer can resume from the point of failure.
Logging: Keep detailed logs of the transfer process to monitor for errors and to have an audit trail.
Monitoring: Use monitoring tools to keep track of the transfer status, performance, and any anomalies.
5. Automation and Scheduling
Automation: Automate the batch transfer process to minimize manual intervention and reduce the risk of human error.
Scheduling: Schedule transfers during off-peak hours to minimize the impact on network performance and to ensure timely updates.
6. Testing and Validation
Test Transfers: Conduct test transfers to ensure that the process works as expected and that the data integrity is maintained.
Validation: Validate the transferred data to ensure it matches the source data and that no data loss or corruption occurred during the transfer.
7. Scalability
Scalable Solutions: Choose solutions and tools that can scale with your data volume. This is especially important as your data grows over time.
Parallel Transfers: If dealing with very large datasets, consider parallel transfers to speed up the process.
8. Cost Management
Cost Analysis: Monitor and analyze the cost of data transfer, including network costs and cloud storage costs.
Optimization: Optimize the transfer process to reduce unnecessary data movement and storage, thereby reducing costs.
9. Documentation and Governance
Documentation: Document the transfer process, including the tools used, the schedule, and any specific configurations.
Governance: Ensure compliance with data governance policies and regulations, particularly if dealing with sensitive or regulated data.
