---
title: "A Table for files - Iceberg"
collection: teaching
type: "Lakehouse"
permalink: /teaching/LakeHouse-Play-Table-Iceberg
date: 2024-07-01
venue: "Iceberg, Spark"
date: 2024-06-01
location: "Docker"
---

The objective of this demonstration is to highlight how we can seemlessly manage tables and contrast it to the issues we had hive.

It took a while trying to get Iceberg to work. I had to deal with way to many jar dependencies. I tried and tried and it was taking more time than I planned and had over a Saturday.
So I did take the easy way out by reusing images which were packaged and ready to go. Follow along the post by [Alex Merced](https://alexmercedcoder.medium.com/creating-a-local-data-lakehouse-using-spark-minio-dremio-nessie-9a92e320b5b3) for the docker set up.

I used Docker Desktop, becaue it is easy to use and I do not have to recall how to use it.

The docker-compose.yml uses the dremio image which has all the Spark related dependencies already packaged. I will try to build one in the future when I have the time. Minio (Which I love) for storage, Nessie for Catalog services and the Sparknotebook.

    services:
      dremio:
        platform: linux/x86_64
        image: dremio/dremio-oss:latest
        ports:
          - 9047:9047
          - 31010:31010
          - 32010:32010
        container_name: dremio
      minioserver:
        image: minio/minio
        ports:
          - 9000:9000
          - 9001:9001
        environment:
          MINIO_ROOT_USER: minioadmin
          MINIO_ROOT_PASSWORD: minioadmin
        container_name: minio
        command: server /data --console-address ":9001"
      spark_notebook:
        image: alexmerced/spark33-notebook
        ports:
          - 8888:8888
        volumes:
          - ./data:/data  # Mounting the host directory ./data to /data in the container
        env_file: .env
        container_name: notebook
      
      nessie:
        image: projectnessie/nessie
        container_name: nessie
        ports:
          - "19120:19120"
    networks:
      default:
        name: iceberg_env
        driver: bridge


First we would need to setup a bucket in Minio with Access Keys which are used by Spark to integrate with Minio. Its is an empty bucket.

![image](https://github.com/user-attachments/assets/ddb94b8d-4f3c-4664-b591-f5ca00e6f51b)

# Code

## Setup

```python
import pyspark
from pyspark.sql import SparkSession
import os
import logging
from pyspark import SparkConf
from pyspark import SparkContext
from pyspark.sql.types import StructType, StructField, LongType, DoubleType, StringType

logging.basicConfig(level=logging.INFO, format="%(asctime)s - %(name)s - %(levelname)s - %(message)s")
logger = logging.getLogger("MyIcebergSparkJob")

# Define sensitive variable. This setup was taken seetup. Need to be cautious about changing the Jars because Spark, Hadoop dependecies can be tricky to deal with.
NESSIE_URI = "http://nessie:19120/api/v1" # This is the URI of Nessie Catalog which was installed via Docker
WAREHOUSE = "s3a://lakehousetables/" # The bucket which we had created for the lakehouse.
AWS_ACCESS_KEY_ID = "qv6vvVhpOFY5crpQTjca" # The access key id of Minio created via the Minio UI
AWS_SECRET_ACCESS_KEY = "YLSitr5op9dkU6TmBL3EPgskRAOyIlAuXODVdxMG" # The access key of Minio created via the Minio UI
AWS_S3_ENDPOINT = "http://minioserver:9000" # The Minio Server as installed via Docker
AWS_REGION = "us-east-1" # this is the default of the simulated S3 environment

# Set environment variables for MinIO and AWS
os.environ["NESSIE_URI"] = NESSIE_URI 
os.environ["WAREHOUSE"] = WAREHOUSE
os.environ["AWS_ACCESS_KEY_ID"] = AWS_ACCESS_KEY_ID
os.environ["AWS_SECRET_ACCESS_KEY"] = AWS_SECRET_ACCESS_KEY
os.environ["AWS_S3_ENDPOINT"] = AWS_S3_ENDPOINT
os.environ["AWS_REGION"] = AWS_REGION
os.environ["MINIO_REGION"] = AWS_REGION
os.environ["AWS_DEFAULT_REGION"] = AWS_REGION

conf = (
    pyspark.SparkConf()
    .setAppName('app_name')
    .set('spark.jars.packages', 'org.apache.iceberg:iceberg-spark-runtime-3.3_2.12:1.3.1,org.projectnessie.nessie-integrations:nessie-spark-extensions-3.3_2.12:0.67.0,software.amazon.awssdk:bundle:2.17.178,software.amazon.awssdk:url-connection-client:2.17.178')
    .set('spark.sql.extensions', 'org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions,org.projectnessie.spark.extensions.NessieSparkSessionExtensions')
    .set('spark.sql.catalog.icebergmanagedplay', 'org.apache.iceberg.spark.SparkCatalog')
    .set('spark.sql.catalog.icebergmanagedplay.uri', NESSIE_URI)
    .set('spark.sql.catalog.icebergmanagedplay.ref', 'main')
    .set('spark.sql.catalog.icebergmanagedplay.authentication.type', 'NONE')
    .set('spark.sql.catalog.icebergmanagedplay.catalog-impl', 'org.apache.iceberg.nessie.NessieCatalog')
    .set('spark.sql.catalog.icebergmanagedplay.s3.endpoint', AWS_S3_ENDPOINT)
    .set('spark.sql.catalog.icebergmanagedplay.s3.region', AWS_REGION)  # Set AWS region for the catalog
    .set('spark.sql.catalog.icebergmanagedplay.warehouse', WAREHOUSE)
    .set('spark.sql.catalog.icebergmanagedplay.io-impl', 'org.apache.iceberg.aws.s3.S3FileIO')
    .set('spark.hadoop.fs.s3a.access.key', AWS_ACCESS_KEY_ID)
    .set('spark.hadoop.fs.s3a.secret.key', AWS_SECRET_ACCESS_KEY)
    .set('spark.hadoop.fs.s3a.endpoint', AWS_S3_ENDPOINT)  # Ensure the endpoint is set correctly
    .set('spark.hadoop.fs.s3a.path.style.access', 'true')
    .set('spark.hadoop.fs.s3a.connection.ssl.enabled', 'false')
    .set('spark.hadoop.fs.s3a.region', AWS_REGION)
)

# Start Spark Session
spark = SparkSession.builder.config(conf=conf).getOrCreate()
print("Spark Running")

# Set Hadoop configuration for S3 within SparkContext
def load_config(spark_context):
    spark_context._jsc.hadoopConfiguration().set("fs.s3a.access.key", AWS_ACCESS_KEY_ID)
    spark_context._jsc.hadoopConfiguration().set("fs.s3a.secret.key", AWS_SECRET_ACCESS_KEY)
    spark_context._jsc.hadoopConfiguration().set("fs.s3a.endpoint", AWS_S3_ENDPOINT)
    spark_context._jsc.hadoopConfiguration().set("fs.s3a.connection.ssl.enabled", "false")
    spark_context._jsc.hadoopConfiguration().set("fs.s3a.path.style.access", "true")
    spark_context._jsc.hadoopConfiguration().set("fs.s3a.region", AWS_REGION)

load_config(spark.sparkContext)
```
## Ingest Table with predefined Schema

Using the ERP data which we had used from the parquet [demonstration](https://nuneskris.github.io/talks/Parquet-BestPracticeDemo). 

```python
from pyspark.sql.types import StructType, StructField, LongType, DoubleType, StringType, IntegerType
# Defining the schema
schema = StructType([
    StructField('SALESORDERID', IntegerType(), True),
    StructField('SALESORDERITEM', IntegerType(), True),
    StructField('PRODUCTID', StringType(), True),
    StructField('NOTEID', StringType(), True),
    StructField('CURRENCY', StringType(), True),
    StructField('GROSSAMOUNT', IntegerType(), True),
    StructField('NETAMOUNT', DoubleType(), True),
    StructField('TAXAMOUNT', DoubleType(), True),
    StructField('ITEMATPSTATUS', StringType(), True),
    StructField('OPITEMPOS', StringType(), True),
    StructField('QUANTITY', IntegerType(), True),
    StructField('QUANTITYUNIT', StringType(), True),
    StructField('DELIVERYDATE', IntegerType(), True)])

# Read data from the mounted directory
df = spark.read.csv("/data/SalesOrderItems.csv", header=True, inferSchema=True)
# Create Iceberg table "nyc.taxis_large" from RDD
df.write.mode("overwrite").saveAsTable("icebergmanagedplay.SalesOrderItems")
# Query table row count
count_df = spark.sql("SELECT COUNT(*) AS cnt FROM icebergmanagedplay.SalesOrderItems")
total_rows_count = count_df.first().cnt
logger.info(f"Total Rows for SalesOrderItems Data: {total_rows_count}")
```
 - MyIcebergSparkJob - INFO - Total Rows for SalesOrderItems Data: 1930

### What Happened with the data?

1. Iceberg created a logical table around the data in Storage (Minio-S3). 
![image](https://github.com/user-attachments/assets/67ec78de-6528-4a2a-8686-f037758d0b69)

2. Iceberg created 2 folders. One for the data and one for the metadata.
![image](https://github.com/user-attachments/assets/a83e9989-17de-46a3-a889-867709a109a6)

3. Iceberg automatically converts the data into Parquet and manages the same.
![image](https://github.com/user-attachments/assets/8a14ccb7-1e56-4018-b8c1-45751b352b20)

4. Iceberg has the all important metadata.json and the manifest data within the metadata folder.
![image](https://github.com/user-attachments/assets/a9ff63c8-9826-4b54-8447-68c7a63c36ab)

![image](https://github.com/user-attachments/assets/1fceb15e-bfd1-4fa6-839a-679d7c7df436)

# Schema Evolution
The current table:
```python
spark.sql("USE icebergmanagedplay")
tables = spark.sql("SHOW TABLES")
tables.show()
```
<img width="354" alt="image" src="https://github.com/user-attachments/assets/746a9946-b9ee-4409-9612-3f7b4cc1e651">


```python
spark.sql("DESCRIBE TABLE EXTENDED icebergmanagedplay.SalesOrderItems").show()
```
![Uploading image.png…]()


## 1. Column Name Change
```python
# Query table row count where we think all the values are null
count_notnull_OPITEMPOS_df = spark.sql("SELECT COUNT(*) AS cnt FROM icebergmanagedplay.SalesOrderItems where OPITEMPOS is not null")
total_nonnull_rows_count = noteid_count_df.first().cnt
logger.info(f"Total Rowsfor SalesOrderItems Data where OPITEMPOS is not null: {total_nonnull_rows_count}")
```
MyIcebergSparkJob - INFO - Total Rowsfor SalesOrderItems Data where OPITEMPOS is not null: 0

```python
# Rename column "OPITEMPOS" in icebergmanagedplay.SalesOrderItems to "OPITEM_POS"
spark.sql("ALTER TABLE icebergmanagedplay.SalesOrderItems RENAME COLUMN OPITEMPOS TO OPITEM_POS")
# Add description to the new column "OPITEM_POS"
spark.sql(
    "ALTER TABLE icebergmanagedplay.SalesOrderItems ALTER COLUMN OPITEM_POS COMMENT 'This Column is modified and we will also delete it.'")
spark.sql("DESCRIBE TABLE EXTENDED icebergmanagedplay.SalesOrderItems").show()
```
+------------------+---------+--------------------+
|          col_name|data_type|             comment|
+------------------+---------+--------------------+
|      SALESORDERID|      int|                    |
|    SALESORDERITEM|      int|                    |
|         PRODUCTID|   string|                    |
|            NOTEID|   string|                    |
|          CURRENCY|   string|                    |
|       GROSSAMOUNT|      int|                    |
|         NETAMOUNT|   double|                    |
|         TAXAMOUNT|   double|                    |
|     ITEMATPSTATUS|   string|                    |
|        OPITEM_POS|   string|This Column is mo...|
|          QUANTITY|      int|                    |
|      QUANTITYUNIT|   string|                    |
|      DELIVERYDATE|      int|                    |
|                  |         |                    |
|    # Partitioning|         |                    |
|   Not partitioned|         |                    |
|                  |         |                    |
|# Metadata Columns|         |                    |
|          _spec_id|      int|                    |
|        _partition| struct<>|                    |
+------------------+---------+--------------------+
only showing top 20 rows

## 2. Droping a column
```python
spark.sql("ALTER TABLE icebergmanagedplay.SalesOrderItems DROP COLUMN OPITEM_POS")
spark.sql("DESCRIBE TABLE EXTENDED icebergmanagedplay.SalesOrderItems").show()
```




