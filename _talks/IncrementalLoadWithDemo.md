---
title: "Incremental (Append and Deduplicate) Load with Airbyte Demo"
collection: talks
permalink: /talks/IncrementalLoadWithDemo
date: 2024-03-01
venue: "Delta"
date: 2024-06-01
location: "Airbyte"
---
<img width="666" alt="image" src="https://github.com/user-attachments/assets/344cb01e-204a-44a3-9b02-a1aef9e4099c">

An "Incremental Append + Deduped Load" is a data integration pattern commonly used in ETL to ensure that duplicate records are avoided at the target database and are maintained at the source table too. This is used when we want to add data directly into a staging layer.
* Append: Adding only the new or changed records from the source system to the target system.
* Deduplication: Ensuring that any duplicate records that might have been introduced during the append process are removed or handled appropriately.

Incremental Append involves extracting only the new or modified records from the source system. This can be achieved by typically using a cursor timestamp audit column. Using a timestamp column (e.g., last_modified or updated_at) to filter and extract only the records that have been updated or created since the last load. Then we use the difference (delta) between the current state and the last state of the data to capture the delta. We could potentially also use CDC tools or techniques to identify and extract only the records that have changed since the last extraction but it can add some complexity.

Once the new and updated records are appended to the target system, deduplication ensures that no duplicate records exist. Deduplication can be done by using primary keys to enforce uniqueness. The we would need to use a MERGE Statement to combine new data with existing data, ensuring that duplicates are handled based on specified conditions. Finally we would use a SQL window functions to identify and remove duplicates, keeping only the latest record based on a timestamp or version column.

SCD1 (Slowly Changing Dimension Type 1) is a suitable use case for the Incremental Append + Deduped Load pattern. SCD1 is used to manage changes in a dimension table where the changes overwrite the existing records. This means that when a change occurs in the source data, the corresponding record in the dimension table is updated with the new information. We need to load only the new or updated records from the source system. This can be achieved by identifying the records that have changed since the last load using a timestamp or some change tracking mechanism. After loading the new and updated records, you need to ensure that the dimension table contains the latest version of each record. This involves removing any older versions of the records that have been updated.

The objective of this demo is to be able to query a source table in Postgres, perform an extract from the source table and load in a desination analytics environment. However this would to most importantly perform only delta changes.
* Full Load
* Insert New Records
* Update Change Records
* Update With Delete Change

# Setup

Please refer a Sanbox demo to set up [Postgres, Airbyte and Snowflake](https://nuneskris.github.io/teaching/Postgres-Airbyte-S3).

<img width="666" alt="image" src="https://github.com/user-attachments/assets/741fff01-aac9-47ae-861f-b03fc9ce80b5">


## Configuring the source.

<img width="666" alt="image" src="https://github.com/user-attachments/assets/b305513c-ae76-47cc-98ee-03b5bd77e4c9">

## Destination
Airbyte created a table in Snowflake with the source columns along with some additional columns.
<img width="666" alt="image" src="https://github.com/user-attachments/assets/d8739752-77d8-4fd4-951c-31ecc5d5f0c9">

## Configuring the Stream Cursor.
A cursor is the value used to track whether a record extracted in an incremental sync. We will be using CHANGEAT for this.

We would need to configure the cursor which will be used to handle the delta updates.
<img width="666" alt="image" src="https://github.com/user-attachments/assets/6e2b09cf-dcfd-45f0-baf9-0f690e443c9a">

## Data Setup

We will leverage the table from the previous demo. But I have upadted the table to use a primarykey.
```sql
ALTER TABLE SALES_ORDER ADD PRIMARY KEY (SALESORDERID)
```
<img width="666" alt="image" src="https://github.com/user-attachments/assets/a023a772-3731-4388-a1aa-224d0007a773">

I had used a python script to load the CSV using the currentdatetime for the CHANGEAT AND CREATEAT DATETIME.

Code to INSERT DATA into postgres using current_datetime = datetime.now().strftime('%Y-%m-%d %H:%M:%S') for CHANGEAT AND CREATEAT DATETIME.

```python
import pandas as pd
import psycopg2
from datetime import datetime

# PostgreSQL credentials
db_username = 'krisnunes'
db_password = ''
db_host = 'localhost'
db_port = '5432'
db_name = 'krisnunes'

# CSV file path
csv_file_path = '/Users/krisnunes/Study/python/DataEngineering/ETL/Collect/Ingest/Postgress/SalesOrdersTimeStamp5.csv'

# Read the CSV file into a DataFrame
df = pd.read_csv(csv_file_path)

# Create a connection to the PostgreSQL database
try:
    connection = psycopg2.connect(
        user=db_username,
        password=db_password,
        host=db_host,
        port=db_port,
        database=db_name
    )
    cursor = connection.cursor()

    # Insert data into the table
    for index, row in df.iterrows():
        # Get the current datetime
        current_datetime = datetime.now().strftime('%Y-%m-%d %H:%M:%S')
        cursor.execute("""
            INSERT INTO sales_order (
                SALESORDERID, CREATEDBY, CREATEDAT, CHANGEDBY, CHANGEDAT, 
                FISCVARIANT, FISCALYEARPERIOD, NOTEID, PARTNERID, SALESORG, 
                CURRENCY, GROSSAMOUNT, NETAMOUNT, TAXAMOUNT, 
                LIFECYCLESTATUS, BILLINGSTATUS, DELIVERYSTATUS
            ) VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s)
        """, (
            row['SALESORDERID'], row['CREATEDBY'], current_datetime, row['CHANGEDBY'], current_datetime, 
            row['FISCVARIANT'], row['FISCALYEARPERIOD'], row['NOTEID'], row['PARTNERID'], row['SALESORG'], 
            row['CURRENCY'], row['GROSSAMOUNT'], row['NETAMOUNT'], row['TAXAMOUNT'], 
            row['LIFECYCLESTATUS'], row['BILLINGSTATUS'], row['DELIVERYSTATUS']
        ))
    connection.commit()

except Exception as error:
    print(f"Error while connecting to PostgreSQL: {error}")

finally:
    if connection:
        cursor.close()
        connection.close()
        print("PostgreSQL connection is closed")
```

<img width="666" alt="image" src="https://github.com/user-attachments/assets/5f42fca9-fa2d-4a02-87cc-1cdfa7ab9f68">

<img width="666" alt="image" src="https://github.com/user-attachments/assets/0646dc76-f3f1-405a-861e-5347ee04a416">


# Demo Run
## 1. Full Load
The FULL LOAD should move all the data in the table. All the 670 rows was syced by Airbyte.

<img width="666" alt="image" src="https://github.com/user-attachments/assets/b8291e60-8633-484d-9038-3eaae566dd4f">

### Validating the change
There were 670 records also created in Snowflake table.

<img width="666" alt="image" src="https://github.com/user-attachments/assets/52d39966-e569-413a-a146-7d583c5a54f7">

Below is a sample data of the table

<img width="666" alt="image" src="https://github.com/user-attachments/assets/1012d328-85a8-42ca-95db-97a164f1c215">

We can see there is somemetadata also added by airbyte.

<img width="666" alt="image" src="https://github.com/user-attachments/assets/ecefaf43-15c1-4556-be55-9def483eefe6">

## 2. Insert New Records

We will be inserting 2 new records.

<img width="666" alt="image" src="https://github.com/user-attachments/assets/d4a4f3b7-076b-4eec-93ac-eeaf0fef1272">

After running the insert query.

<img width="666" alt="image" src="https://github.com/user-attachments/assets/0684a094-9d5a-4859-ad92-9f9cd8e6864f">

### Running Airbyte Sync
<img width="666" alt="image" src="https://github.com/user-attachments/assets/18030ce5-9e85-45a0-905b-5fb49483b6f2">

### Validating Data in Snowflake
Running a count we see there are 2 new records added taking the count to 672.
<img width="666" alt="image" src="https://github.com/user-attachments/assets/f018a012-9709-4c78-a6cf-2a6348b5a73e">

We can query the metadata and we can see that there was a new run.

<img width="666" alt="image" src="https://github.com/user-attachments/assets/67d378bf-8f06-49d8-a9cc-90f83c851bd6">

We are able to get the run synid and query the records which were inserted.

<img width="666" alt="image" src="https://github.com/user-attachments/assets/a06a9646-3a64-406c-b652-f7d6e3da4a42">


## 3. Update Change Records

#### Updating a record which was 
My original dataset had 2

Code to UDPATE DATA into postgres using current_datetime = datetime.now().strftime('%Y-%m-%d %H:%M:%S') for CHANGEAT DATETIME.
```python
import pandas as pd
import psycopg2
from datetime import datetime

# PostgreSQL credentials
db_username = 'krisnunes'
db_password = ''
db_host = 'localhost'
db_port = '5432'
db_name = 'krisnunes'

# CSV file path
csv_file_path = '/Users/krisnunes/Study/python/DataEngineering/ETL/Collect/Ingest/Postgress/SalesOrdersTimeStamp4.csv'

# Read the CSV file into a DataFrame
df = pd.read_csv(csv_file_path)

# Create a connection to the PostgreSQL database
try:
    connection = psycopg2.connect(
        user=db_username,
        password=db_password,
        host=db_host,
        port=db_port,
        database=db_name
    )
    cursor = connection.cursor()

    # Insert data into the table
    for index, row in df.iterrows():
        # Get the current datetime
        current_datetime = datetime.now().strftime('%Y-%m-%d %H:%M:%S')
        cursor.execute("""
           UPDATE sales_order SET
                CREATEDBY = %s,
                CREATEDAT = %s,
                CHANGEDBY = %s,
                CHANGEDAT = %s,
                FISCVARIANT = %s,
                FISCALYEARPERIOD = %s,
                NOTEID = %s,
                PARTNERID = %s,
                SALESORG = %s,
                CURRENCY = %s,
                GROSSAMOUNT = %s,
                NETAMOUNT = %s,
                TAXAMOUNT = %s,
                LIFECYCLESTATUS = %s,
                BILLINGSTATUS = %s,
                DELIVERYSTATUS = %s
            WHERE SALESORDERID = %s
        """, (
            row['CREATEDBY'], row['CREATEDAT'], row['CHANGEDBY'], current_datetime, 
            row['FISCVARIANT'], row['FISCALYEARPERIOD'], row['NOTEID'], row['PARTNERID'], row['SALESORG'], 
            row['CURRENCY'], row['GROSSAMOUNT'], row['NETAMOUNT'], row['TAXAMOUNT'], 
            row['LIFECYCLESTATUS'], row['BILLINGSTATUS'], row['DELIVERYSTATUS'],
            row['SALESORDERID']  # The identifier to find the correct record
        ))
    connection.commit()

except Exception as error:
    print(f"Error while connecting to PostgreSQL: {error}")

finally:
    if connection:
        cursor.close()
        connection.close()
        print("PostgreSQL connection is closed")

```

We will be updating the below 4 rows.
```sql
select * from sales_order where salesorderid in (500000100,500000101,500000102,500000103, 500000000, 500000001);
```

<img width="612" alt="image" src="https://github.com/user-attachments/assets/c86bd623-49e8-445b-b73a-860d3e843ba4">

<img width="613" alt="image" src="https://github.com/user-attachments/assets/33f9cdc1-4971-4b19-afbd-3b9c7185bd04">

![image](https://github.com/user-attachments/assets/01dd2234-0e02-4b1a-a20b-9a594916af45)

![image](https://github.com/user-attachments/assets/cc1dfc77-f496-413b-8522-a0972cc15b93)

![image](https://github.com/user-attachments/assets/5921a24d-04d6-47e7-be75-d83a5fcbe456)

![image](https://github.com/user-attachments/assets/8941d1f4-c99f-42f5-be35-dd91dff55c42)


## 4. Delete

The delete will use a Note column to indicate that the row is deleted and perform a row update.

<img width="612" alt="image" src="https://github.com/user-attachments/assets/23e30936-a7ab-4b9e-9904-534f477c36bd">

![image](https://github.com/user-attachments/assets/9bbd3fe4-a500-4c1d-8726-2f74a4c10a04)

![image](https://github.com/user-attachments/assets/a6ea6fec-7d85-43ce-a27d-4a27df12702c)






