---
title: "Key Features and Advantages of Apache Iceberg"
collection: publications
permalink: /publication/Hive-vs-Iceberg
excerpt: ''
venue: 'Lakehouse'
tags:
  - Store
---

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
Note: I have a detailed demo on [Spark ETL on Iceberg](https://nuneskris.github.io/teaching/LakeHouse-Play-Table-Iceberg-ETL) on this very topic.

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
(1, 'John Doe', 30, 50000.0),
(2, 'Jane Smith', 25, 60000.0),
(3, 'Alice Johnson', 35, 70000.0)
""")

df_before = spark.sql("SELECT * FROM glue_catalog.db.employee")
df_before.show()
```

| id|        name|age|  salary|
|--|-----------|--|-------|
|  1|    John Doe| 30|50000.0 |
|  2|  Jane Smith| 25|60000.0 |
|  3|Alice Johnson| 35|70000.0 |


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
(4, 'Bob Brown', 40, 80000.0),
(5, 'Charlie Green', 45, 90000.0)
""")
# Query data after changing the column type
df_after = spark.sql("SELECT * FROM glue_catalog.db.employee")
df_after.show()
```

| id|         name|age|  salary|
|---|------------|--|------|
|  1|   John Doe  | 30|50000.0 |
|  2| Jane Smith  | 25|60000.0 |
|  3|Alice Johnson| 35|70000.0 |
|  4|  Bob Brown  | 40|80000.0 |
|  5|Charlie Green| 45|90000.0 |


As we have seen when the column type is changed, Iceberg updates the schema metadata but does not rewrite the data files. The old data files are read with the old schema, and new data files are written with the new schema. This is possible because Iceberg tracks the schema version for each file, allowing it to handle mixed schema versions seamlessly. Iceberg's advanced schema evolution capabilities mean that you typically do not need to rewrite the entire table when changing a column type. Instead, Iceberg manages schema changes through metadata updates, ensuring efficient and backward-compatible schema evolution. This is a significant advantage over traditional systems like Hive, where such changes often require costly and time-consuming data rewrites.
----------

Hidden Partitioning:

Iceberg: Allows for hidden partitioning, meaning partition columns do not need to be part of the schema, and partitioning logic can change without rewriting data.
Hive: Requires explicit partition columns in the schema, making schema changes and partitioning logic changes more complex.
ACID Transactions:

Iceberg: Provides full ACID transaction support including snapshot isolation, allowing for complex multi-row updates and deletes.
Hive: While Hive supports ACID transactions, they are often less performant and more complex to manage compared to Iceberg.
Time Travel:

Iceberg: Supports time travel, enabling queries on historical snapshots of data, which is useful for debugging, auditing, and reproducing reports.
Hive: Does not natively support time travel capabilities.
Data Compaction:

Iceberg: Automatically handles data compaction in the background, optimizing the data layout and reducing read amplification.
Hive: Requires manual intervention and scheduling for compaction, which can lead to maintenance overhead.
Efficient File Management:

Iceberg: Manages data files at the table level, allowing for better control over file sizes, fewer small files, and optimized read performance.
Hive: Can suffer from small file problems, especially in scenarios with frequent updates and deletes.
Query Performance:

Iceberg: Optimized for query performance with features like column-level stats and file-level pruning, which minimize the amount of data scanned during queries.
Hive: Generally less optimized for these aspects, leading to potentially higher query latencies for large datasets.
Data Layout Optimizations:

Iceberg: Supports data layout optimization features like automatic partitioning and clustering.
Hive: Data layout optimizations are more manual and require explicit configuration and management.
Built-in Support for Multiple Engines:

Iceberg: Provides native support for multiple compute engines, including Apache Spark, Flink, Presto, Trino, and more.
Hive: Primarily optimized for the Hive query engine, though integrations with other engines exist but are not as seamless.
Snapshot Isolation:

Iceberg: Offers snapshot isolation, allowing concurrent reads and writes without locking, which enhances performance in concurrent environments.
Hive: ACID transactions in Hive might require more locking, impacting performance.
Data Format Agnostic:

Iceberg: Supports multiple data formats (e.g., Parquet, Avro, ORC) with consistent management and performance optimizations.
Hive: Primarily optimized for ORC format, though it supports other formats.
Conclusion
Apache Iceberg provides a modern approach to data lake management with features that simplify data management, improve performance, and support evolving data use cases. Its advanced capabilities, such as schema evolution, hidden partitioning, and time travel, make it a powerful tool for handling large-scale analytics and data warehousing tasks that go beyond the traditional capabilities of Apache Hive.
