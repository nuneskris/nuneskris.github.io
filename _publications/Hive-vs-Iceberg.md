---
title: "Modern Table Formats"
collection: publications
permalink: /publication/Hive-vs-Iceberg
excerpt: 'A comparative analysis between Hive and Iceberg'
venue: 'Lakehouse'
tags:
  - Store
---

Note: I have a detailed demo on [Spark ETL on Iceberg](https://nuneskris.github.io/teaching/LakeHouse-Play-Table-Iceberg-ETL) on this very topic.

# Schema Evolution
Our analytics systems are built iteratively and very often we start with a base model and then refine/expand the model in subsequently. Schema evolution refers to the capability to adapt to changes in the schema (structure) of the data over time. This includes adding, deleting, or modifying columns in a table without requiring a complete rewrite of the table or data loss.

## How did we evolve Schema in Hive?
Schema changes can be more cumbersome and often require full table rewrites or manual schema migration steps.

Lets start with the table employee where age is an INT

```sql
CREATE TABLE employee (
  id INT,
  name STRING,
  age INT,
  salary FLOAT
) STORED AS PARQUET;
-- Insert data
INSERT INTO employee VALUES (1, 'Kris', 42), (2, 'Hamlet', 25);
```

Lets say we want to change employee age from INT to BIGINT.
### Step 1: Create a new table
```sql
CREATE TABLE employee_temp (
  id INT,
  name STRING,
  age BIGINT, -- Changed from INT to BIGINT
  salary FLOAT
) STORED AS PARQUET;
```

### Step 2: Insert Data with Type Casting:
```sql
INSERT INTO employee_temp
SELECT id, name, CAST(age AS BIGINT), salary
FROM employee;
```
### Step 2:  Drop the Original Table and remanem the new temp table
```sql
DROP TABLE employee;
ALTER TABLE employee_temp RENAME TO employee;
```

## How do we evolve Schema in Iceberg?

When you change a column type in Iceberg, the new schema version is recorded, and the readers and writers understand how to interpret the data based on the schema version. Here is an example of how to change a column type in Iceberg using PySpark:

Here’s a similar example where we change the salary column from FLOAT to DOUBLE in an Iceberg table. This process will include creating the table, inserting data, changing the column type, and querying data before and after the change.

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder \
    .config("spark.sql.catalog.glue_catalog", "org.apache.iceberg.spark.SparkCatalog") \
    .config("spark.sql.catalog.glue_catalog.type", "hadoop") \
    .config("spark.sql.catalog.glue_catalog.warehouse", "s3://your-bucket/warehouse") \
    .getOrCreate()

spark.sql("""
CREATE TABLE glue_catalog.db.employee (
  id INT,
  name STRING,
  age INT,
  salary FLOAT
) USING iceberg
LOCATION 's3://your-bucket/warehouse/employee'
""")
```
Insert data
```python
spark.sql("""
INSERT INTO glue_catalog.db.employee VALUES
(1, 'Kris Nunes', 30, 50000.0),
(2, 'Tom Hardy', 25, 60000.0),
(3, 'Iris Murdoch', 35, 70000.0)
""")

df_before = spark.sql("SELECT * FROM glue_catalog.db.employee")
df_before.show()
```

| id|        name|age|  salary|
|--|-----------|--|-------|
|  1|    Kris Nunes| 30|50000.0 |
|  2|  Tom Hardy| 25|60000.0 |
|  3|Iris Murdoch| 35|70000.0 |


### Simple alter statement will Alter table and its data. 
```python
# Change the column type
spark.sql("""
ALTER TABLE glue_catalog.db.employee ALTER COLUMN salary TYPE DOUBLE
""")
```
Now lets test and see what happened.
```python
# Insert more data
spark.sql("""
INSERT INTO glue_catalog.db.employee VALUES
(4, 'Mag Atwood', 40, 80000.0),
(5, 'Paul Scott', 45, 90000.0)
""")
# Query data after changing the column type
df_after = spark.sql("SELECT * FROM glue_catalog.db.employee")
df_after.show()
```

| id|         name|age|  salary|
|---|------------|--|------|
|  1|   Kris Nunes  | 30|50000.0 |
|  2| Tom Hardy  | 25|60000.0 |
|  3|Iris Murdoch| 35|70000.0 |
|  4|  Mag Atwood  | 40|80000.0 |
|  5|Paul Scott| 45|90000.0 |


As we have seen when the column type is changed, Iceberg updates the schema metadata but does not rewrite the data files. The old data files are read with the old schema, and new data files are written with the new schema. This is possible because Iceberg tracks the schema version for each file, allowing it to handle mixed schema versions seamlessly. Iceberg's advanced schema evolution capabilities mean that you typically do not need to rewrite the entire table when changing a column type. Instead, Iceberg manages schema changes through metadata updates, ensuring efficient and backward-compatible schema evolution. This is a significant advantage over traditional systems like Hive, where such changes often require costly and time-consuming data rewrites.
----------

# Time Travel

Time Travel in the context of data management refers to the ability to query and access historical versions of data at different points in time. This feature allows users to view the state of data as it was at a specific moment in the past, which is valuable for auditing, debugging, and recovering from accidental changes.

***Historical Queries*** is a ability of Time travel which enables querying data as it existed at a particular snapshot or timestamp. This is useful for understanding changes over time, auditing, and rolling back to a previous state. Data systems that support time travel often take ***periodic snapshots*** of the data. Each snapshot represents the state of the data at a specific time, allowing users to access different versions of the data. Time travel is typically implemented by ***versioning data***. Each version corresponds to a snapshot, and users can specify which version to query based on its identifier or timestamp.

### The usecases are just beyond analytics usecases.
The ability supports recovery from errors or unintended changes. If data is modified or deleted by mistake, users can query the previous state and restore the data if needed. It supports auditing by allowing users to review historical changes and understand how the data evolved over time. This is useful for compliance and traceability.

## An Example

### Setup: let us  create an Iceberg table.

```python
from pyspark.sql import SparkSession

# Initialize Spark session with Iceberg configurations
spark = SparkSession.builder \
    .config("spark.sql.catalog.glue_catalog", "org.apache.iceberg.spark.SparkCatalog") \
    .config("spark.sql.catalog.glue_catalog.type", "hadoop") \
    .config("spark.sql.catalog.glue_catalog.warehouse", "s3://your-bucket/warehouse") \
    .getOrCreate()

# Create the Iceberg table
# Perform some updates
spark.sql("""
    UPDATE spark_catalog.default.employee
    SET salary = 2500.0
    WHERE id = 1
""")

# Insert more data
spark.sql("""
    INSERT INTO spark_catalog.default.employee VALUES
    (4, 'David', 3000.0)
""")
```

###  Setup: Query the data in the table.
```python
df_initial = spark.sql("SELECT * FROM glue_catalog.db.time_travel_example")
df_initial.show()
```

| id|      name| salary|
|---|----------|--------|
|  1|  John Doe|50000.0|
|  2|Jane Smith|60000.0|

###  Test Time Travel: Update Data
```python
spark.sql("""
UPDATE glue_catalog.db.time_travel_example
SET salary = 55000.0
WHERE id = 1
""")

# Insert additional data
spark.sql("""
INSERT INTO glue_catalog.db.time_travel_example VALUES
(3, 'Alice Johnson', 70000.0)
""")
```

### Query Historical Data Using Time Travel
Iceberg supports querying historical snapshots of the table. You can use the versionAsOf option to query data as of a specific snapshot version or timestamp.

```python
# List all snapshots to find the version number or timestamp
snapshots_df = spark.sql("SELECT * FROM glue_catalog.db.time_travel_example.snapshots")
snapshots_df.show()
```
Assume we get the following snapshot IDs:

Snapshot 1: 994875543435455698272
Snapshot 2: 657465496797328734634
Snapshot 3: 233438345678434562938

```python
# Query the table at the first snapshot
spark.sql("SELECT * FROM spark_catalog.default.employee VERSION AS OF 994875543435455698272").show()

# Query the table at the second snapshot
spark.sql("SELECT * FROM spark_catalog.default.employee VERSION AS OF 657465496797328734634").show()
```

Snapshot 1:

| id|   name|salary|
|---|-------|------|
|  1|  Alice|1000.0|
|  2|    Bob|1500.0|
|  3|Charlie|2000.0|

Snapshot 2: The salary is updated to 2500

| id|   name|salary|
|---|-------|------|
|  1|  Alice|2500.0|
|  2|    Bob|1500.0|
|  3|Charlie|2000.0|

Snapshot 3: New row is added

| id|   name|salary|
|---|-------|------|
|  1|  Alice|2500.0|
|  2|    Bob|1500.0|
|  3|Charlie|2000.0|
|  4|  David|3000.0|

# Data Format Agnostic:
Iceberg supports multiple data formats (e.g., Parquet, Avro, ORC) with consistent management and performance optimizations. Hive primarily optimized for ORC format, though it supports other formats.

I have a detailed [demonstration](https://nuneskris.github.io/talks/Parquet-BestPracticeDemo) on the best practices on Parquet.

# Hidden Partitioning:

Hidden partitioning in Iceberg refers to the concept where the partitioning scheme is defined once at table creation and then abstracted away from the user during data manipulation and querying. Unlike traditional partitioning schemes, where you must be aware of and handle partition columns explicitly in your queries, Iceberg allows you to work with the data without needing to know the specifics of the partitioning strategy.

* Users do not need to include partition columns in their queries explicitly. Iceberg handles the partition pruning automatically, simplifying query writing.
* Iceberg uses the partitioning information to optimize query execution under the hood. This means that partition pruning and data skipping are handled transparently, leading to better performance without additional user effort.
* Iceberg supports schema evolution without breaking the partitioning scheme. This flexibility allows you to add, remove, or rename columns without having to redefine the partitioning strategy or reorganize your data.

## How Does Hive Handle Partition
Hive handles partitions differently from Iceberg. In Hive, partitioning is explicitly managed and visible to the user. You need to specify the partition columns both when creating the table and when querying the data. This explicit partitioning scheme can make querying more cumbersome, as users must be aware of the partition structure to write efficient queries In Hive, partitioning is a technique to divide a large table into smaller, more manageable pieces. Each partition corresponds to a specific value or set of values of one or more columns. This helps in reducing the amount of data scanned during queries, leading to improved query performance.

### Example of Hive Partitioning

#### Create a Hive Table with Partitions
The PARTITIONED BY clause specifies the columns to be used for partitioning. In this case, the employee table is partitioned by year and month.
```sql
CREATE TABLE employee (
    id INT,
    name STRING,
    department STRING,
    salary FLOAT
)
PARTITIONED BY (year INT, month INT)
STORED AS PARQUET;
```
#### Load Data into the Hive Table
When loading data into a partitioned table, you need to specify the partition values explicitly. When we insert data we need to define the partition values also.
Data is stored in directories corresponding to the partition values (e.g., year=2023/month=1).
```sql
-- Load data into partition (year=2023, month=01)
INSERT INTO employee PARTITION (year=2023, month=1)
VALUES
    (1, 'Kris', 'HR', 70000),
    (2, 'Juan', 'Engineering', 80000);

-- Load data into partition (year=2023, month=02)
INSERT INTO employee PARTITION (year=2023, month=2)
VALUES
    (3, 'Nunes', 'Marketing', 75000),
    (4, 'Roach', 'Finance', 85000);
```
#### Query the Partitioned Data
When querying a partitioned table in Hive, you should include the partition columns in the query for optimal performance.
You need to include the partition columns in the WHERE clause to take advantage of partition pruning. This helps in scanning only the relevant partitions, reducing the amount of data read during the query.
```sql
-- Query data for January 2023
SELECT id, name, department, salary
FROM employee
WHERE year = 2023 AND month = 1;

-- Query data for all months in 2023
SELECT id, name, department, salary
FROM employee
WHERE year = 2023;
```

## How Does Iceberg Handle Partition

## Setup
Ensure you have the necessary dependencies and configurations for PySpark and Iceberg.
```python
from pyspark.sql import SparkSession

spark = SparkSession.builder \
    .appName("IcebergExample") \
    .config("spark.sql.catalog.my_catalog", "org.apache.iceberg.spark.SparkCatalog") \
    .config("spark.sql.catalog.my_catalog.type", "hadoop") \
    .config("spark.sql.catalog.my_catalog.warehouse", "s3://my-bucket/my-warehouse") \
    .getOrCreate()

# Define the schema for the table
schema = """
    id INT,
    data STRING,
    category STRING,
    event_date DATE
"""

# Create the Iceberg table with hidden partitioning
spark.sql(f"""
    CREATE TABLE my_catalog.db.my_table (
        {schema}
    ) USING iceberg
    PARTITIONED BY (bucket(16, id), days(event_date))
""")
```
The PARTITIONED BY clause defines how the table is partitioned. In this case, we use bucket(16, id) and days(event_date). This means the data will be partitioned by bucketing the id column into 16 buckets and by day for the event_date column.

## Now let us insert data
You can insert data into the table without worrying about how the data will be partitioned. Iceberg handles this based on the defined partitioning scheme
```python
from pyspark.sql.functions import lit
from datetime import date

# Sample data
data = [
    (1, "data1", "A", date(2023, 1, 1)),
    (2, "data2", "B", date(2023, 1, 2)),
    (3, "data3", "A", date(2023, 1, 3)),
    (4, "data4", "C", date(2023, 1, 4)),
    (5, "data5", "B", date(2023, 1, 5))
]

# Create a DataFrame
df = spark.createDataFrame(data, ["id", "data", "category", "event_date"])

# Write the DataFrame to the Iceberg table
df.writeTo("my_catalog.db.my_table").append()
```
Step 4: Query the Data
When querying the data, you do not need to include partition columns in your query explicitly. Iceberg will automatically use the partitioning information to optimize the query execution. When filtering by event_date, Iceberg will use the partitioning scheme to minimize the amount of data scanned, resulting in more efficient queries.
```python

# Query the table without knowing the partitioning scheme
df = spark.read.format("iceberg").load("my_catalog.db.my_table")
df.show()

# Filter by event_date to demonstrate hidden partitioning efficiency
df.filter("event_date = '2023-01-01'").show()
```

# Overhead of Deletes and Updates in Hive

Hive’s implementation of deletes and updates introduces several overheads and complexities due to its design and reliance on underlying Hadoop Distributed File System (HDFS) architecture. Here’s a detailed look at the specific challenges and overheads associated with delete and update operations in Hive:

```sql
CREATE TABLE employee (
    id INT,
    name STRING,
    salary INT
)
CLUSTERED BY (id) INTO 3 BUCKETS
STORED AS ORC
TBLPROPERTIES ('transactional'='true');
--- Insert Initial Data:
INSERT INTO employee VALUES (1, 'Alice', 1000), (2, 'Bob', 1200);
--- Update Operation:
UPDATE employee SET salary = 1100 WHERE id = 1;
--- Delete Operation:
DELETE FROM employee WHERE id = 2;
```


Delta Files Creation:
Mechanism: When a delete or update operation is performed in Hive, it doesn't modify the existing data files directly. Instead, Hive creates delta files that record the changes (deletions or new versions of rows).
Overhead: These delta files accumulate over time, leading to a proliferation of small files that can degrade query performance and increase storage costs.
Compaction Requirements:
Minor and Major Compaction: Hive requires periodic compaction to merge these delta files with the original data files. Minor compaction merges small delta files into larger delta files, while major compaction merges all delta files with the base files.
Resource-Intensive: Compaction is a resource-intensive process that can consume significant CPU and I/O resources, impacting the performance of the Hive cluster. Compaction needs to be scheduled and managed carefully to avoid impacting regular query performance.
Concurrency Issues:
Locking Mechanism: Hive uses a locking mechanism to manage concurrent access to tables during delete and update operations. This can lead to contention and reduced concurrency, especially in environments with high write throughput.
Transaction Management: Hive’s transaction management system is not as sophisticated as modern ACID-compliant systems, leading to potential bottlenecks and reduced performance under heavy concurrent load.
Read Performance Degradation:
File Scanning: When querying a table with many delta files, Hive needs to scan both the base files and the delta files, which increases the read latency. The more delta files there are, the more files Hive needs to scan, which can significantly degrade query performance.
Complexity in Query Execution: The presence of delta files complicates query execution plans, as the query engine needs to reconcile the base and delta files to provide the current view of the data.
Manual Maintenance:
Scheduled Compaction: Administrators need to manually schedule and manage compaction jobs to ensure that the delta files are periodically merged. This requires careful planning and monitoring to avoid disruptions.
Housekeeping Tasks: Ongoing housekeeping tasks, such as managing old delta files and monitoring the health of the transaction log, add operational overhead.
Limited Support for Deletes:
Partition-Level Deletes: Hive’s delete operations are generally more efficient when applied at the partition level rather than the row level. Row-level deletes are less efficient and can lead to a large number of small delta files.
Configuration Complexity: Properly configuring Hive for efficient delete operations requires tuning various settings and parameters, which can be complex and error-prone.




Compaction Example:

Minor Compaction:
sql
Copy code
ALTER TABLE employee COMPACT 'MINOR';
Major Compaction:
sql
Copy code
ALTER TABLE employee COMPACT 'MAJOR';
Impact of Compaction:
Minor Compaction: Merges small delta files into larger delta files to reduce the number of files, but doesn’t merge with base files.
Major Compaction: Merges all delta files with the base files to create a new set of base files, significantly reducing the number of files but consuming more resources.
Summary
Hive's approach to handling deletes and updates through delta files and compaction introduces significant overheads. The need for periodic compaction to maintain performance, the accumulation of small files, and the manual management of these processes add complexity and resource demands. These overheads can impact the overall performance and scalability of Hive in environments with frequent delete and update operations.


# ACID Transactions:
Iceberg: Provides full ACID transaction support including snapshot isolation, allowing for complex multi-row updates and deletes.
Hive: While Hive supports ACID transactions, they are often less performant and more complex to manage compared to Iceberg.



# Efficient File Management:
Iceberg: Manages data files at the table level, allowing for better control over file sizes, fewer small files, and optimized read performance.
Hive: Can suffer from small file problems, especially in scenarios with frequent updates and deletes.

# Query Performance:
Iceberg: Optimized for query performance with features like column-level stats and file-level pruning, which minimize the amount of data scanned during queries.
Hive: Generally less optimized for these aspects, leading to potentially higher query latencies for large datasets.

# Data Layout Optimizations:
Iceberg: Supports data layout optimization features like automatic partitioning and clustering.
Hive: Data layout optimizations are more manual and require explicit configuration and management.

# Built-in Support for Multiple Engines:
Iceberg: Provides native support for multiple compute engines, including Apache Spark, Flink, Presto, Trino, and more.
Hive: Primarily optimized for the Hive query engine, though integrations with other engines exist but are not as seamless.

# Snapshot Isolation:
Iceberg: Offers snapshot isolation, allowing concurrent reads and writes without locking, which enhances performance in concurrent environments.
Hive: ACID transactions in Hive might require more locking, impacting performance.

Conclusion
Apache Iceberg provides a modern approach to data lake management with features that simplify data management, improve performance, and support evolving data use cases. Its advanced capabilities, such as schema evolution, hidden partitioning, and time travel, make it a powerful tool for handling large-scale analytics and data warehousing tasks that go beyond the traditional capabilities of Apache Hive.
