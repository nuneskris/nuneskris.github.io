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


