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

Note: I have a demo simple demo on launching [Hive on AWS EMR Serverless](https://nuneskris.github.io/teaching/Hive-EMR-Serverless). 


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
One of the simplest methods was to read the data into an external processing framework (like MapReduce, Spark, or Pig), apply the necessary transformations (updates or deletes), and then write the processed data back into a new table. Afterward, the old table would be dropped, and the new table would be renamed to replace the old one. We would create a temporary table with the desired modifications and then swap the temporary table with the original table. This approach involved creating a new table, applying updates or deletes during the data load process, and then replacing the original table with the modified table. Partitions were also leveraged for this but it also came with much effort. 

There is a an update to Hive, when a delete or update operation is performed in Hive, it doesn't modify the existing data files directly. Instead, Hive creates delta files that record the changes (deletions or new versions of rows). Hive also requires periodic compaction to merge the small delta files with the original data files which very resource intensive.

## Iceberg
Update and deletes are very simple in ICEBERG. I have [demonstration](https://nuneskris.github.io/talks/Slowly-Changing-Dimensions) on SCD2 dimensions using ICEBERG and spark were I perform mutliple update opeations.

```python
# Create initial DataFrame
data = [(1, 'Kris Nunes', 50000), (2, 'Nunes Kris', 60000)]
df = spark.createDataFrame(data, ["id", "name", "salary"])

# Write to Iceberg table
df.writeTo("my_catalog.db.employee").append()

```

| id|    name| salary|
|--|-------|-------|
|  1|Kris Nunes|50000.0|
|  2|Nunes Kris|60000.0|

```python
from pyspark.sql.functions import col

# Load the table in the 
employee_df = spark.table("my_catalog.db.employee")

# Update data
updated_df = employee_df.withColumn("salary", 
              col("salary").when(col("id") == 1, 70000).otherwise(col("salary")))

# Overwrite the table with updated data
updated_df.writeTo("my_catalog.db.employee").overwritePartitions()
```

| id|    name| salary|
|--|-------|-------|
|  1|Kris Nunes|70000.0|
|  2|Nunes Kris|60000.0|

```python
# Filter out the row to delete and overwrite the table
filtered_df = employee_df.filter(col("id") != 2)
filtered_df.writeTo("my_catalog.db.employee").overwritePartitions()
```

| id|    name| salary|
|--|-------|-------|
|  1|Kris Nunes|70000.0|

# Other important Reasons
Iceberg provides full ACID transaction support including snapshot isolation, allowing for complex multi-row updates and deletes. While Hive supports ACID transactions, they are often less performant and more complex to manage compared to Iceberg. Hive is an old technology and Iceberg is optimized for query performance with features like column-level stats and file-level pruning, which minimize the amount of data scanned during queries. Iceberg provides native support for multiple compute engines, including Apache Spark, Flink, Presto, Trino, and more. Apache Iceberg provides a modern approach to data lake management with features that simplify data management, improve performance, and support evolving data use cases. Its advanced capabilities, such as schema evolution, hidden partitioning, and time travel, make it a powerful tool for handling large-scale analytics and data warehousing tasks that go beyond the traditional capabilities of Apache Hive.
