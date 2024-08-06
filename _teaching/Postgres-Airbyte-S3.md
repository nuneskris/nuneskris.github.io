---
title: "Collect: Airbyte Extract And Load to S3"
collection: teaching
type: "Lakehouse"
permalink: /teaching/Postgres-Airbyte-S3
venue: "Airflow"
date: 2024-06-01
---

<img width="700" alt="image" src="https://github.com/user-attachments/assets/f0286140-40f4-4396-810e-d05d4d028e40">


# Set Up

## Install Postgres
* For interst of time, I installed Postgress via an installer

```sql
CREATE TABLE sales_order (
    SALESORDERID BIGINT PRIMARY KEY,
    CREATEDBY BIGINT,
    CREATEDAT TIMESTAMP,
    CHANGEDBY BIGINT,
    CHANGEDAT TIMESTAMP,
    FISCVARIANT VARCHAR,
    FISCALYEARPERIOD BIGINT,
    NOTEID VARCHAR,
    PARTNERID BIGINT,
    SALESORG VARCHAR,
    CURRENCY VARCHAR,
    GROSSAMOUNT BIGINT,
    NETAMOUNT DOUBLE PRECISION,
    TAXAMOUNT DOUBLE PRECISION,
    LIFECYCLESTATUS VARCHAR,
    BILLINGSTATUS VARCHAR,
    DELIVERYSTATUS VARCHAR
);
```

Insert data from csv to postgress
```python
import pandas as pd
import psycopg2

# PostgreSQL credentials
db_username = 'xxxxxx'
db_password = ''
db_host = 'localhost'
db_port = '5432'
db_name = 'xxxxxx'

# CSV file path
csv_file_path = '/XXXXXXXX/Study/python/DataEngineering/ETL/Collect/Ingest/Postgress/SalesOrdersTimeStamp.csv'

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
        cursor.execute("""
            INSERT INTO sales_order (
                SALESORDERID, CREATEDBY, CREATEDAT, CHANGEDBY, CHANGEDAT, 
                FISCVARIANT, FISCALYEARPERIOD, NOTEID, PARTNERID, SALESORG, 
                CURRENCY, GROSSAMOUNT, NETAMOUNT, TAXAMOUNT, 
                LIFECYCLESTATUS, BILLINGSTATUS, DELIVERYSTATUS
            ) VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s)
        """, (
            row['SALESORDERID'], row['CREATEDBY'], row['CREATEDAT'], row['CHANGEDBY'], row['CHANGEDAT'], 
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

## Install Airbyte
****clone Airbyte from GitHub****
```console
git clone --depth=1 https://github.com/airbytehq/airbyte.git
```

****switch into Airbyte directory****
```console
cd airbyte
```

****start Airbyte****
```console
./run-ab-platform.sh
```

Using the default config
* Host: http://localhost:8000/
* User: airbyte and password is password

## S3

* Create a user with access to the Bucket. Generate access key
* Create a folder.

![image](https://github.com/user-attachments/assets/0ca2cc32-1b45-4c0a-8ad3-cbeedef0ec12)

# Configure Airbyte

## Source: Postgress
![image](https://github.com/user-attachments/assets/e534e0c4-4a22-4d73-b5c1-333ed60b6aed)

I am using the basic update.
![image](https://github.com/user-attachments/assets/75f7a4ca-2454-4959-874d-51f530e67923)

## Target: S3
![image](https://github.com/user-attachments/assets/80d4ffd6-5507-4980-8b85-58ffe9a7897c)

# Running a sync

![image](https://github.com/user-attachments/assets/7794fe7f-74a5-47b7-8ca6-16d88a10c9e8)

# Validating in S3
![image](https://github.com/user-attachments/assets/296a9883-872b-4148-a5ba-cb7f32edcf63)




