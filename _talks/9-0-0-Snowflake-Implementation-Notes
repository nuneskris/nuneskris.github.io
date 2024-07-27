---
title: "Snowflake Implementation Notes"
collection: talks
type: "Tutorial"
permalink: /talks/Snowflake-Implementation-Notes
date: 2020-03-01
---

# Virtual Warehouses

* Standard
* Snowpark-optimized
* A warehouse provides the required resources, such as CPU, memory, and temporary storage, to perform operations in a Snowflake session such as SQL SELECT Queries and DML statements [DELETE, INSERT, UPDATE, COPY INTO (both loading and unloading)]
* XSMALL default when we use CREATE WAREHOUSE command
* XLARGE default when we use Web Interface
* per-second billing with minimum 60 second
* Increasing the size of a warehouse does not always improve data loading performance. 
* The size of a warehouse can impact the amount of time required to execute queries submitted to the warehouse, particularly for larger, more complex queries. In general, query performance scales with warehouse size because larger warehouses have more compute resources available to process queries.
* Larger is not necessarily faster for small, basic queries.
* Auto-suspension and Auto-resumption
    * By default, auto-suspend and auto-resume is enabled. 
* STATEMENT_QUEUED_TIMEOUT_IN_SECONDS
* STATEMENT_TIMEOUT_IN_SECONDS
* If queries are queuing more than desired, another warehouse can be created and queries can be manually redirected to the new warehouse. In addition, resizing a warehouse can enable limited scaling for query concurrency and queuing; however, warehouse resizing is primarily intended for improving query performance.
* To enable fully automated scaling for concurrency, Snowflake recommends multi-cluster warehouses, which provide essentially the same benefits as creating additional warehouses and redirecting queries, but without requiring manual intervention.
* Multi-cluster warehouses are an Enterprise Edition feature.
* When a session is initiated in Snowflake, the session does not, by default, have a warehouse associated with it. Until a session has a warehouse associated with it, queries cannot be submitted within the session.
* A default warehouse can be specified when creating or modifying the user, either through the web interface or using CREATE USER/ALTER USER.

Enterprise:
- Standard +
- Multi-cluster
- 90 day time travel
- Materialized views
- Search Optimization
- Dynamic Data Masking
- External Data tokenization
- Annual Rekeying of external data

Business Critical
- Enterprise +
- HIPPA
- PCI
- Tri-secret customer managed keys
- Private Link for AWS, Azure, GCP
- Database failover and failback

VPS
- BC +
- Customer Dedicated Virtual Servers where encrypt key in in-house
- Customer Dedicated Metadata store

Multicluster
* Maximum number of clusters, greater than 1 (up to 10).
* Maximized vs AutoScale
Scaling Policy
* Standard (default): 
    * Prevents/minimizes queuing by favoring starting additional clusters over conserving credits.
    * Warehouse Shuts Down: After 2 to 3 consecutive successful checks performed at 1 min intervals
    * Starts: The first cluster starts immediately when either a query is queued or the system detects that there’s one more query than the currently-running clusters can execute. (Waits 20 seconds)
* Economy: 
    * Conserves credits by favoring keeping running clusters fully-loaded rather than starting additional clusters, which may result in queries being queued and taking longer to complete.
    * Warehouse Shuts Down: After 5 to 6 consecutive successful checks performed at 1 min intervals
    * Starts: Only if the system estimates there’s enough query load to keep the cluster busy for at least 6 minutes

Considerations
* Per second billing
* The minimum billing charge for provisioning compute resources is 1 minute (i.e. 60 seconds).
* warehouse resizing is not intended for handling concurrency issues; instead, use additional warehouses to handle the workload or use a multi-cluster warehouse

Query Acceleration Service
The query acceleration service does this by offloading portions of the query processing work to shared compute resources that are provided by the service.
* Ad hoc analytics.
* Workloads with unpredictable data volume per query.
* Queries with large scans and selective filters.
* Analyze 
    * SYSTEM$ESTIMATE_QUERY_ACCELERATION view: function can help determine if a previously executed query might benefit from the query acceleration service. 
    *  QUERY_ACCELERATION_ELIGIBLE view: identify the queries and warehouses that might benefit the most from the query acceleration service.
* ENABLE_QUERY_ACCELERATION = TRUE when creating a warehouse
* QUERY_ACCELERATION_MAX_SCALE_FACTOR property with default set as 8
* Query History view
    * QUERY_ACCELERATION_BYTES_SCANNED
    * QUERY_ACCELERATION_PARTITIONS_SCANNED
    * QUERY_ACCELERATION_UPPER_LIMIT_SCALE_FACTOR
    * 
Ineligible Queries
* There are no filters or aggregation (i.e. GROUP BY). The query acceleration service is currently unable to accelerate such queries.
* The filters are not selective enough. Alternatively, the GROUP BY expression has a high cardinality.
* There are not enough partitions. The benefit of query acceleration would be offset by the latency in acquiring additional servers for the service if there are not enough partitions to scan.
* The query includes a LIMIT clause. However, a LIMIT clause with an ORDER BY clause is supported.


### Monitoring Load

Load Monitoring Chart
* Peak Query Performance
* Slow Query Performance
* Excessive Credit Usage

### QUERY_HISTORY View
Excessive Credit Usage
If the chart shows recurring time periods when the warehouse was running and consuming credits, but the total query load was less than 1 for substantial periods of time,
Warehouse Performance
throughput and latency

Snowpark-optimized Warehouses
- provide 16x memory per node compared to a standard Snowflake virtual warehouse
- ML Training and UDF
- WAREHOUSE_TYPE = 'SNOWPARK-OPTIMIZED';

Databases, Tables and Views

Table Structure

Micro Partition
* Each micro-partition contains between 50 MB and 500 MB of uncompressed data 
* Micro-partitioning is automatically performed on all Snowflake tables. Tables are transparently partitioned using the ordering of the data as it is inserted/loaded.
* Columns are stored and compressed independently within micro-partition
* Delete all is only a meta-data operation
* DML (Update, Delete, Merge) leverages Micro partitions for efficiency
* Pruning: Do not scan partitions which do not where the data cannot have matching values
* Snowflake does not prune micro-partitions based on a predicate with a subquery, even if the subquery results in a constant
Data Clustering
* Clustering Depth
    * A table with no micro-partitions (i.e. an unpopulated/empty table) has a clustering depth of 0
    * 
Clustering Keys & Clustered Tables
A clustering key is a subset of columns in a table (or expressions on a table) that are explicitly designated to co-locate the data in the table in the same micro-partitions. 
* When is it needed:
    * You require the fastest possible response times, regardless of cost.
    * Your improved query performance offsets the credits required to cluster and maintain the table.
* Benefits of Defining Clustering Keys
    * Improved scan efficiency in queries by skipping data that does not match filtering predicates.
    * Better column compression than in tables with no clustering. This is especially true when other columns are strongly correlated with the columns that comprise the clustering key.
    * After a key has been defined on a table, no additional administration is required, unless you chose to drop or modify the key. All future maintenance on the rows in the table (to ensure optimal clustering) is performed automatically by Snowflake.
* Considerations
    * The table contains a large number of micro-partitions. Typically, this means that the table contains multiple terabytes (TB) of data.
    * The queries can take advantage of clustering. Typically, this means that one or both of the following are true:
        * The queries are selective. In other words, the queries need to read only a small percentage of rows (and thus usually a small percentage of micro-partitions) in the table.
        * The queries sort the data. (For example, the query contains an ORDER BY clause on the table.)
    * A high percentage of the queries can benefit from the same clustering key(s). In other words, many/most queries select on, or sort on, the same few column(s).
* Re-clustering
    * Automatic
Automatic Clustering
* Full Control: ALTER TABLE … SUSPEND / RESUME RECLUSTER
* Non-blocking DML

Temporary and Transient Tables
Temporary Tables
- Temporary tables only exist within the session in which they were created and persist only for the remainder of the session
- Can name with the same name within the schema and it will take precedence
- We can have time-travel on a temporary table of 1 day
- No fail safe
Transient Tables
- Transient tables are similar to permanent tables with the key difference that they do not have a Fail-safe period (which is 7 days for Permanent tables).
- Transient and temporary tables can have a Time Travel retention period of either 0 or 1 day.
- 
External Tables
* query data stored in an external stage as if the data were inside a table in Snowflake
* The external stage is not part of Snowflake, so Snowflake does not store or manage the stage.
* Recommend in Parquet for parallel reads

Views
* definition for a view cannot be updated
* Changes to a table are not automatically propagated to views created on that table
* Views are read-only 
Non-materialized View
- Regular views do not cache data, and therefore cannot improve performance by caching
Materialized View
- When to use
    - Query results contain a small number of rows and/or columns relative to the base table
    - Query results contain results that require significant processing (Semi-Structured Data, Calculation of Aggregates)
    - The query is on an external table
    - The view’s base table does not change frequently
    - Supports clustering
Secure Views
- For a non-secure view, the view definition is visible to other users. The definition of a secure view is only exposed to authorized users
- For a non-secure view, internal optimizations can indirectly expose data.
- Secure views should not be used for views that are defined solely for query convenience, such as views created to simplify queries for which users do not need to understand the underlying data representation. 
- Secure views can execute more slowly than non-secure views.
-  trade-off between data privacy/security and query performance.
- IS_SECURE column in the Information Schema and Account Usage views identifies whether a view is secure.
- The internals of a secure view are not exposed in Query Profile (in the web interface). 
- secure views, Snowflake does not expose the amount of data scanned

Views
Advantages of views
- More modular code
- Allows access to a subset of a table
- Materialized views can improve performance
Limitations: 
- Views cannot be altered
- read-only
- Changes to the table are not propogated to the view.
* Non-materialized views (usually simply referred to as “views”)
* Materialized Views
    * When to use [Obvious]
        * When external tables are used and to avoid repeating going to an external table
        * Very small subset of the entire table is needed
        * Query has results which require significant processing
    * When not to use [Obvious]
    * A background service updates the materialized view after changes are made to the base table. Data is always current.
    * Limitations
        * Cannot include another view or a user-defined function or window functions
        * Can query only a single table
        * Functions used in a materialized view must be deterministic. For example, using CURRENT_TIME or CURRENT_TIMESTAMP is not permitted.
    * Cost: Storage plus compute for maintenance
    * We can use clustering in materialized views
* Secure Views
    * Not to be used just for convenience as they are slower than normal views
    * When to use
        * Hide the table definitions
        * Indirect access to data can be avoided
    * is_secure in select of information_schema or account_usage can be used to identify if secure

Table Design Consideration
- Referential integrity constraints in Snowflake are informational and, with the exception of NOT NULL, not enforced. However they provide valuable documentation
- Do not use character for time dates
- Though column size does not impact the performance, it allows to detect issues
- Use Variant for semi-structured data

Consideration

Table Design
- Snowflake recommends choosing a date or timestamp data type rather than a character data type. 
- Referential integrity constraints in Snowflake are informational and, with the exception of NOT NULL, not enforced. 
- However it helps for documentation and during BI tool integration.
- Clustering Key
    - Specifying a clustering key is not necessary for most tables.
    - Consider specifying when
        - Query Profiler indicated too much time is spent no scanning
        - Order of Data loaded does not match the dimension it is queried the most.
- Snowflake compresses column data effectively; therefore, creating columns larger than necessary has minimal impact on the size of data tables.
- If you are not sure yet what types of operations you want perform on your semi-structured data, Snowflake recommends storing the data in a VARIANT column for now.
- FLATTEN: For better pruning and less storage consumption, Snowflake recommends flattening your object and key data into separate relational columns if your semi-structured data includes:
- FLATTEN: Non-native values such as dates and timestamps are stored as strings when loaded into a VARIANT column, so operations on these values could be slower and also consume more space than when stored in a relational column with the corresponding data type.
- Currently, it is not possible to change a permanent table to a transient table using the ALTER TABLE command. The TRANSIENT property is set at table creation and cannot be modified and VICE-VERSA
    - Create a new table, copy grants 

Cloning
- Individual external named stages can be cloned. 
- Internal (i.e. Snowflake) named stages cannot be cloned.
- Data Sequence: If the table is cloned, cloned table refers the source sequence. If the database or 
- 
Data Storage

Data Types

Data Loading
- Snowflake refers to the location of data files in cloud storage as a stage.
- External Stage: Managed externally: Create Stage
- Internal Stage
    - User Stage: staged and managed by a single user. Cannot be dropped or altered
        * User stages are referenced using @~; e.g. use LIST @~ to list the files in a user stage.
        * LIST @~;
        * Unlike named stages, user stages cannot be altered or dropped.
        * User stages do not support setting file format options. Instead, you must specify file format and copy options as part of the COPY INTO <table> command.
    - Table Stage: staged and managed by one or more users but only loaded into a single table. 
        * Convenient way multiple users to have access to the files
        * Table stages have the same name as the table; e.g. a table named mytable has a stage referenced as @%mytable.
        * LIST @%mytable;
        * Unlike named stages, table stages cannot be altered or dropped.
        * Table stages do not support transforming data while loading it (i.e. using a query as the source for the COPY command).
        * It is in implicit object tied to the table 
    - Named Stage: A named internal stage is a database object created in a schema.
        * LIST @my_stage;
        * greatest degree of flexibility for data loading
        * Users with the appropriate privileges on the stage can load data into any table.
        * Security functionality available on internal stage
- Copy Command
    - Bulk and Continuous (Snowpipe)
    - Simple Transformations
Considerations
    - The number of load operations that run in parallel cannot exceed the number of data files to be loaded. To optimize the number of parallel operations for a load, we recommend aiming to produce data files roughly 100-250 MB (or larger) in size compressed.
    - Loading very large files (e.g. 100 GB or larger) is not recommended.
    - Aggregate smaller files to minimize the processing overhead for each file.
    - Continuous Load will happen every minute so accordingly breakup data if it is too large (100 - 250 MB) or create smaller files when the data is sparse.
    - Loading large data sets can affect query performance. We recommend dedicating separate warehouses for loading and querying operations to optimize performance for each.
    - Organizing Data by Path
Bulk Loading
Execute PUT using the SnowSQL client or Drivers to upload (stage) local data files into an internal stage.
Get and PUT do not support external stage. We need to use cloud client for them.
PUT commas cannot be executed from the worksheet
Validating Your Data
- VALIDATION_MODE
- ON_ERROR
Monitoring Files Staged Internally
- Metadata maintained
    - File name
    - File size (compressed, if compression was specified during upload)
    - LAST_MODIFIED date, i.e. the timestamp when the data file was initially staged or when it was last modified, whichever is later
- Snowflake retains historical data for COPY INTO commands executed within the previous 14 days.
- Use the LIST command to view the status of data files that have been staged.
- Monitor the status of each COPY INTO <table> command on the History page of the Classic Console.
- Use the VALIDATE function to validate the data files you’ve loaded and retrieve any errors encountered during the load.
- Use the LOAD_HISTORY Information Schema view to retrieve the history of data loaded into tables using the COPY INTO command.
Managing Data Files
- Staged files can be deleted from a Snowflake stage
    - PURGE copy option in the COPY INTO <table> command.
    * After the load completes, use the REMOVE command to remove the files in the stage
Snowpipe
	Snowpipe enables loading data from files as soon as they’re available in a stage. This means you can load data from files in micro-batches, making it available to users within minutes, rather than manually executing COPY statements on a schedule to load larger batches. The data is loaded according to the COPY statement defined in a referenced pipe.

* Automating Snowpipe using cloud messaging
* Calling Snowpipe REST endpoints
How Is Snowpipe Different from Bulk Data Loading?
- Authentication
    - Bulk data load: Relies on the security options supported by the client for authenticating and initiating a user session.
    - Snowpipe: When calling the REST endpoints: Requires key pair authentication with JSON Web Token (JWT). JWTs are signed using a public/private key pair with RSA encryption.
- Load History
    - Stored in the metadata of the target table for 64 days.
    - Stored in the metadata of the pipe for 14 days. 
- Compute Resources
    - Bulk data load: Requires a user-specified warehouse to execute COPY statements.
    - Snowpipe: Uses Snowflake-supplied compute resources.
- Cost
    - Bulk data load: Billed for the amount of time each virtual warehouse is active.
    - Snowpipe: Billed according to the compute resources used in the Snowpipe warehouse while loading the files.
Recommended Load File Size
- staging files once per minute.
Auto Ingest: Leverages Cloud Messaging. (Notification of the filename)>
* 14 days. > 14 days the pipe becomes stale
* More than 14 days is best effort.
Rest End Point
* insertFiles
* insertReport:
    * last 10mins with 10K max
* loadHistoryScan: 
    * 2 points in time with 10K max
* The notification is stored indefinitely and need to be purged manually.

Error Notifications: Snowpipe can push error notifications to a cloud messaging service
* Snowpipe error notifications only work when the ON_ERROR copy option is set to SKIP_FILE (the default)
* NOTIFICATION_HISTORY table function

Managing Snowpipe
Pipe objects do not support the PURGE copy option. Snowpipe cannot delete staged files automatically when the data is successfully loaded into tables.To remove staged files that you no longer need, we recommend periodically executing the REMOVE command to delete the files. Or configure lifecycle management.



Snowpipe Streaming
The API is intended to complement Snowpipe, not replace it. Use the Snowpipe Streaming API in streaming scenarios where data is streamed via rows (for example, Apache Kafka topics) instead of written to files. The API fits into an ingest workflow that includes an existing custom Java application that produces or receives records. The API removes the need to create files to load data into Snowflake tables and enables the automatic, continuous loading of data streams into Snowflake as the data becomes available.

Continuous Data Pipelines
* Data Loading
    * Snowpipe
    * Snowpipe Streaming
    * Snowflake Connector Kafka
* Continuous Transformation: Dynamic Tables
* Change Data Tracking: Streams
* Recurring Tasks	



Queries

Joins
* Outer Join - all plus matches from the outer
* Inner Join - only matches
* Cross Join - cartesian product (all possible combination
* Natural Join - named match of a column
Subqueries
-- Uncorrelated subquery: An uncorrelated subquery has no such external column references
	SELECT c1, c2 FROM table1 WHERE c1 = (SELECT MAX(x) FROM table2);

-- Correlated subquery: A correlated subquery can be thought of as a filter on the table that it refers to.
	SELECT c1, c2 FROM table1 WHERE c1 = (SELECT x FROM table2 WHERE y = table1.c2);

 Query hierarchical data - Pending
* Recursive CTEs (common table expressions): more flexible and allow a table to be joined to one or more other tables.
* CONNECT BY clauses : allows only self-joins



Data Sharing & Collaboration





Alerts and Notifications - Done

Snowflake Alerts
-  EXECUTE ALERT privilege: only be granted by a user with the ACCOUNTADMIN role.
- SCHEDULED_TIME
- LAST_SUCCESSFUL_SCHEDULED_TIME
- ALTER ALERT myalert SUSPEND
- ALTER ALERT myalert RESUME;

Email Notifications
- SYSTEM$SEND_EMAIL() stored procedure


Security
ORGADMIN —>  
* Creates Accounts
ACCOUNTADMIN —>
*  SYSADMIN + SECURITYADMIN
SECURITYADMIN —> 
* 	CREATE AND MODIFY USERS globally
USERADMIN —>
* Is granted the CREATE USER and CREATE ROLE security privileges.
* Can create users and roles in the account.
* Has OWNERSHIP privileges on the Object (Users).
    * Only the role with the OWNERSHIP privilege on an object (i.e. user or role), or a higher role, can modify the object properties.
SYSADMIN
* Creates warehouses, databases, and all database objects (schemas, tables, etc.).

The only option is to drop and recreate the share. Ownership of a share cannot be granted to another role

Organization
An organization is a first-class Snowflake object that links the accounts owned by your business entity. View, create, and manage all of your accounts across different regions and cloud platforms
ORGADMIN Role


Data Governance
Data Sensitivity & Access Visibility:Object Tagging
* A single table or view object: 50 unique tags.
* All columns combined in a single table or view: 50 unique tags.
* A tag is inherited based on the Snowflake securable object hierarchy.
* Snowflake replicates tags and their assignments within the primary database to the secondary database.
* Sensitive Data Tracking and Resource Usage
* SELECT * FROM SNOWFLAKE.ACCOUNT_USAGE.TAG ORDER BY TAG_NAME;
* TAG_REFERENCES_WITH_LINEAGE to determine all of the objects that have a given tag key and tag value that also includes the tag lineage

# Data Classification
* Analyze: EXTRACT_SEMANTIC_CATEGORIES
* Review:
* Apply: ASSOCIATE_SEMANTIC_CATEGORY_TAGS
* System tags and categories
    * SNOWFLAKE.CORE.SEMANTIC_CATEGORY
        * 
    * SNOWFLAKE.CORE.PRIVACY_CATEGORY
        *  Identifier
        * Quasi-identifier
            *  Sensitive
Data Access Policies
Masking Policies
	Column Level Security
        * Dynamic Data Masking: selectively mask plain-text data in table and view columns at query time.
        * External Tokenization: tokenize data before loading it into Snowflake and detokenize the data at query runtime.

Access History
source columns
projected columns
Columns to determine result (where)

# Column lineage 

# Business Continuity & Data Recovery

# Replication and Failover
* Replication group as read-only
* Failover is a replication group that can also act as a failover and become read-write
* Failover Group, Data protected with Tri-Secret Secure, Account object (other than database and share) replication only available for Business Critical and VPS (virtual private snowflake) editions. They are available in Standard and enterprise

# Data Recovery
## Time Travel
* Snowflake Standard: Default for 1 and can be set to 0
* Snowflake Enterprise: Transient Table: Default 1 and can be set to 0, Permanent can be set from 0 to 90
* When retention period ends, it goes into fail-safe
    * Historical data no longer available for querying
    * Past objects can no longer be cloned
    * Past object which were dropped can no longer be restored.
* DATA_RETENTION_TIME_IN_DAYS 
* MIN_DATA_RETENTION_TIME_IN_DAYS  at the ACCOUNTADMIN
* Can be used with SELECT, CLONE using AT|BEFORE
* Once dropped objects are moved to Fail-safe, you cannot restore them.
## Fail-Safe
* Fail-safe is a data recovery service that is provided on a best effort basis and is intended only for use when all other recovery options have been attempted.
* Fail-safe provides a (non-configurable) 7-day period during which historical data may be recoverable by Snowflake. This period starts immediately after the Time Travel retention period ends. 
* It is for use only by Snowflake to recover data that may have been lost or damaged due to extreme operational failures.
* No User operations allowed
* Transient - 0 days and permanent 7 days.

## Performance Optimization 
The Account Usage schema contains views related to the execution times of queries and tasks.
* QUERY_HISTORY
* WAREHOUSE_LOAD_HISTORY
* TASK_HISTORY

# Optimizing Warehouses for Performance

# Cost Management
There resources contribute to cost
1. Compute Resources
    1. Virtual Warehouse Compute:
        *  Virtual warehouses are user-managed compute resources that consume credits when loading data, executing queries, and performing other DML operations.
        * Snowflake utilizes per-second billing (with a 60-second minimum each time the warehouse starts)
        * How long they are used * what size * number 
        * Warehouses are only billed for credit usage while running. When a warehouse is suspended, it does not use any credits.
        * Suspending and then resuming a warehouse within the first minute results in multiple charges because the 1-minute minimum starts over each time a warehouse is resumed.
        * Resizing a warehouse from 5X-Large or 6X-Large to 4X-Large (or smaller) results in a brief period during which the warehouse is billed for both the new compute resources and the old resources while the old resources are quiesced.
    2. Serverless Compute
        *  Search Optimization, Snowpipe (& Streaming), External Tables, Materialized Views, Query Acceleration, Tasks
        * To keep cost low, automatically resized and scaled up or down by Snowflake as required for each workload.
        * Compute hours rounded to the nearest second
    3. Cloud Services Compute
        * behind-the-scenes tasks such as authentication, metadata management, access control, query caching, optimizer
        * charged only if the daily consumption of cloud services resources exceeds 10% of the daily warehouse usage [daily]
2. Storage Resources
    *  flat rate per terabyte (TB).
    1. Staged File Costs
    2. Database Costs
        * Include historical data maintained for Time Travel. 
        * Snowflake automatically compresses all data stored in tables and uses the compressed file size to calculate the total storage used for an account.
    3. Time Travel and Fail-safe Costs
        * 24 Hr 
        * Snowflake minimizes the amount of storage required for historical data by maintaining only the information required to restore the individual table rows that were updated or deleted. 
        * Storage usage is calculated as a percentage of the table that changed. 
        * Full copies of tables are only maintained when tables are dropped or truncated.
    4. Temporary and Transient Tables Costs
        * Transient and temporary tables can, at most, incur a one day’s worth of storage cost. [They do not have fail safe]
        * Using Temporary and Transient Tables to Manage Storage Costs
            - Temporary tables are dropped when the session in which they were created ends. Data stored in temporary tables is not recoverable after the table is dropped.
            - Historical data in transient tables cannot be recovered by Snowflake after the Time Travel retention period ends. Use transient tables only for data you can replicate or reproduce independently from Snowflake.
            - Long-lived tables, such as fact tables, should always be defined as permanent to ensure they are fully protected by Fail-safe.
            - Short-lived tables (i.e. <1 day), such as ETL work tables, can be defined as transient to eliminate Fail-safe costs.
            - If downtime and the time required to reload lost data are factors, permanent tables, even with their added Fail-safe costs, may offer a better overall solution than transient tables.
    5. Cloning Tables, Schemas, and Databases Costs
        * Snowflake’s zero-copy cloning is extremely useful for creating instant backups that do not incur any additional costs (until changes are made to the cloned object).
        * CDP?
3. Data Transfer Cost
    * Data transfers within the same region are free.
    * Snowflake charges a per-byte fee for data egress when users transfer data 
        * from a Snowflake account into a different region on the same cloud platform 
        * into a completely different cloud platform
	

Explore historical cost using Snowsight, the Snowflake web interface, or by writing queries against views in the ACCOUNT_USAGE and ORGANIZATION_USAGE schemas.
* USAGE_VIEWER — Provides access to a single account in Snowsight and to related views in the ACCOUNT_USAGE schema.
* ORGANIZATION_USAGE_VIEWER — Assuming the current account is the ORGADMIN account, provides access to all accounts in Snowsight and to views in the ORGANIZATION_USAGE schema that are related to cost and usage, but not billing.
*  USAGE_IN_CURRENCY view in the ORGANIZATION_USAGe
###  Monitoring Cost
A resource monitor can be used to monitor credit usage by virtual warehouses and the cloud services needed to support those warehouses. 
Credit quota specifies the number of Snowflake credits allocated to the monitor for the specified frequency interval
* Notify & Suspend
* Notify & Suspend Immediately
* Notify



