---
title: "Collect - Extraction: Batch Transfer"
collection: publications
permalink: /publication/Collect-Data-Extraction-Batch
excerpt: 'Batch Transfer: One of the most common scenarios of extraction. For now atleast'
venue: 'Processing'
date: 2024-05-01
tags:
  - Collect
---

We need to extract data from source applications using simple queries for each table, outputting the results into flat files. These files are then ingested into analytics platforms in the cloud. I have often observed that pipelines make significant mistakes in this subsystem, especially when transferring data from on-premises to the cloud. This is primarily due to the need for coordination between two teams: the application team, which owns the source application, and the data team, which owns the analytics applications.

To address these issues, I propose a framework to build a subsystem architecture that ensures efficiency, reliability, and security.

# Extraction Query
Extraction queries need to be robust and free from failures. Modern business applications typically provide decent data integrity checks before persisting data. A good rule of thumb is to minimize pre-ingest processing and use straightforward queries to extract data in batches, relying on time windows for filtering. However, if processing logic must be applied within the extract query, we need to ensure these queries do not lead to failures.

Here are some common reasons for extraction query failures and their mitigation strategies:

1. ***Syntax Errors from Schema Evolution***: Table schema changes without informing pipeline teams can lead to disruptions. While this is a larger governance issue, we need to design our queries to handle schema changes gracefully.
2. ***Query Memory and CPU Resource Limitations***: Many source applications have resource limits. It is recommended to break queries down into sub-batches to avoid exceeding these limits.
3. ***Disk Space for Extracted Files***: Extracting very large data sets can cause performance degradation or failures due to insufficient disk space for temporary files or output. I have seen major incidents caused by this. It is advisable to delete files from on-premises once they are transferred to the cloud and use the cloud for archiving.
4. ***Query Complexity***: Complex joins, subqueries, heavy aggregation, and grouping operations can lead to performance issues or timeouts. Simplify queries where possible to avoid these issues.
5. ***Uncoordinated System Maintenance Downtimes***: Unexpected maintenance windows during extraction schedules can cause query failures. Regularly coordinate with system administrators to avoid conflicts.
6. ***Permission and Access Issues***: Ensure stable access permissions by using service accounts instead of individual user accounts. Necessary permissions to access data or execute queries can change, or previously granted access can be revoked.
7. ***Data Issues***: While simple extraction queries typically avoid data issues, converting data into compressed formats, especially those that are schema-sensitive, can encounter mismatched data types or unexpected null values causing logic errors.

### Recommendations
Maintain regular coordination between the source application and data teams to plan for any changes. Include exception handling for schema-related constraints to ensure that data encountered is as expected. Automate extraction processes and set up monitoring to quickly identify and resolve issues.

# Efficient Data Transfer
This is very straight forward and the most important architecture decision in collect subsystem. But this is where most of the mistakes are made.
1. Compress data before transfer to reduce the amount of data being sent over the network, which can speed up the transfer and reduce costs. I have a case study on a how multiple delays in pipeline was attributes to not handling move over the network effienciently. Refer [Collect - Extraction: Transfer Compressed Data](https://nuneskris.github.io/publication/Collect-Data-Extraction-Compress).
2. Be carefull with Pull Based APIs: I have seen situations where REST based calls over the network being used espcially with SAP data. They had to go through multiple iterations and breakdown the pull into smaller pulls and coordinate delays. It worked but it was a brittle solution which did not scale. I had recommended to move a simpler push solution atleast for the large tables.
3. Break down large datasets into smaller, manageable chunks to avoid overwhelming the network and to facilitate easier retries in case of failure.

# Security
Encryption: Encrypt data in transit using secure protocols (e.g., TLS) to protect sensitive information from being intercepted.
Authentication: Use strong authentication methods to ensure that only authorized users and systems can initiate data transfers.
Access Controls: Implement strict access controls to limit who can access the data and the transfer mechanisms.

10. Reliability and Monitoring
Retry Mechanism: Implement retry logic to handle transient network issues and ensure that data transfer can resume from the point of failure.
Logging: Keep detailed logs of the transfer process to monitor for errors and to have an audit trail.
Monitoring: Use monitoring tools to keep track of the transfer status, performance, and any anomalies.
11. Automation and Scheduling
Automation: Automate the batch transfer process to minimize manual intervention and reduce the risk of human error.
Scheduling: Schedule transfers during off-peak hours to minimize the impact on network performance and to ensure timely updates.
12. Testing and Validation
Test Transfers: Conduct test transfers to ensure that the process works as expected and that the data integrity is maintained.
Validation: Validate the transferred data to ensure it matches the source data and that no data loss or corruption occurred during the transfer.
13. Scalability
Scalable Solutions: Choose solutions and tools that can scale with your data volume. This is especially important as your data grows over time.
Parallel Transfers: If dealing with very large datasets, consider parallel transfers to speed up the process.
14. Cost Management
Cost Analysis: Monitor and analyze the cost of data transfer, including network costs and cloud storage costs.
Optimization: Optimize the transfer process to reduce unnecessary data movement and storage, thereby reducing costs.
15. Documentation and Governance
Documentation: Document the transfer process, including the tools used, the schedule, and any specific configurations.
Governance: Ensure compliance with data governance policies and regulations, particularly if dealing with sensitive or regulated data.
