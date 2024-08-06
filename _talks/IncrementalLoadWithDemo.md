---
title: "Incremental Load with Airbyte Demo"
collection: talks
permalink: /talks/IncrementalLoadWithDemo
date: 2024-03-01
venue: "Delta"
date: 2024-06-01
location: "Airbyte"
---
<img width="666" alt="image" src="https://github.com/user-attachments/assets/dbdfc4e6-ed1c-41cc-b1ec-f29d7abe7a96">

The objective of this demo is to be able to query a source table in Postgres, perform an extract from the source table and load in a desination analytics environment. However this would to most importantly perform only delta changes.
* Full Load
* Insert New Records
* Update Change Records
* Update With Delete Change

We will leverage audit colum CHANGEAT to capture

# Setup

Please refer a Sanbox demo to set up [Postgres, Airbyte and Snowflake](https://nuneskris.github.io/teaching/Postgres-Airbyte-S3).

<img width="666" alt="image" src="https://github.com/user-attachments/assets/741fff01-aac9-47ae-861f-b03fc9ce80b5">


## Configuring the source.

<img width="666" alt="image" src="https://github.com/user-attachments/assets/b305513c-ae76-47cc-98ee-03b5bd77e4c9">


# Configuring the Stream Cursor.
A cursor is the value used to track whether a record extracted in an incremental sync. We will be using CHANGEAT for this.

We would need to configure the cursor which will be used to handle the delta updates.
<img width="666" alt="image" src="https://github.com/user-attachments/assets/6e2b09cf-dcfd-45f0-baf9-0f690e443c9a">


# Data Setup

We will leverage the table from the previous demo. But I have upadted the table to use a primarykey.
```sql
ALTER TABLE SALES_ORDER ADD PRIMARY KEY (SALESORDERID)
```
![image](https://github.com/user-attachments/assets/a023a772-3731-4388-a1aa-224d0007a773)

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

![image](https://github.com/user-attachments/assets/5f42fca9-fa2d-4a02-87cc-1cdfa7ab9f68)

![image](https://github.com/user-attachments/assets/0646dc76-f3f1-405a-861e-5347ee04a416)


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

