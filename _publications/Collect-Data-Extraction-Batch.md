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

<img width="631" alt="image" src="https://github.com/user-attachments/assets/70bed399-1ce4-4e78-928f-3ea9470b119f">

We need to extract data from source applications using simple queries for each table, outputting the results into flat files. These files are then ingested into analytics platforms in the cloud. I have often observed that pipelines make significant mistakes in this subsystem, especially when transferring data from on-premises to the cloud. This is primarily due to the need for coordination between two teams: the application team, which owns the source application, and the data team, which owns the analytics applications.

To address these issues, I propose a framework to build a subsystem architecture that ensures efficiency, reliability, and security.

# 1. Extraction Query
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

# 2. Efficient Data Transfer
This is very straight forward and the most important architecture decision in collect subsystem. But this is where most of the mistakes are made.
1. Compress data before transfer to reduce the amount of data being sent over the network, which can speed up the transfer and reduce costs. I have a case study on a how multiple delays in pipeline was attributes to not handling move over the network effienciently. Refer [Collect - Extraction: Transfer Compressed Data](https://nuneskris.github.io/publication/Collect-Data-Extraction-Compress).
2. Be carefull with Pull Based APIs: I have seen situations where REST based calls over the network being used espcially with SAP data. They had to go through multiple iterations and breakdown the pull into smaller pulls and coordinate delays. It worked but it was a brittle solution which did not scale. I had recommended to move a simpler push solution atleast for the large tables.
3. Break down large datasets into smaller, manageable chunks to avoid overwhelming the network and to facilitate easier retries in case of failure. Consider parallel transfers to speed up the process.
4. Choose solutions and tools that can scale with your data volume. This is especially important as your data grows over time.

# 3. Security
1. ***Encrypt data in transit*** using secure protocols (e.g., TLS) to protect sensitive information from being intercepted.
2. ***Use strong authentication*** methods to ensure that only authorized users and systems can initiate data transfers.
3. ***Implement strict access controls to limit who can access*** the data and the transfer mechanisms. Utilize Identity and Access Management (IAM) Solutions to manage user identities and access rights centrally. Define and enforce access policies that specify who can access which resources and under what conditions.
4. ***Regularly Audit and Monitor Access*** and maintain detailed logs of authentication attempts, access requests, and data transfers. If we are dealing with sensite data, we would need to monitor logs for suspicious activities and conduct regular audits to ensure compliance with security policies.
5. ***Use service agents*** to connect to source and target of the data transfer, and implement Role-Based Access Control (RBAC) based on ***Least Privilege***. Define roles with specific access rights only to read from source and write into target.
6. ***Use encrypted tokens and API keys*** for authenticating API calls. I have seen this happening but we also need to store tokens and keys securely (e.g., using environment variables or secret management services). Secret Managers provided by cloud services are ideal for ensuring we do not expose passwords or other authenticating details lying around.

# 4. Reliability and Monitoring
1. ***Implement retry logic*** to handle network issues and ensure that data transfer can resume from the point of failure.
3. ***Keep detailed logs*** of the transfer process to monitor for errors and to have an audit trail.
4. ***Use monitoring tools*** to keep track of the transfer status, performance, and any anomalies.
5. ***Conduct test transfers*** to ensure that the process works as expected and that the data integrity is maintained.
6. ***Continously validate*** the transferred data to ensure it matches the source data and that no data loss or corruption occurred during the transfer.
7. ***Document the transfer process***, including the tools used, the schedule, and any specific configurations.

# 5. Automation and Scheduling
1. Automate the batch transfer process to minimize manual intervention and reduce the risk of human error.
2. Schedule transfers during off-peak hours to minimize the impact on network performance and to ensure timely updates.
3. Ensure there are handshakes on whether the 
