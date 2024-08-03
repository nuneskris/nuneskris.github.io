---
title: "TAvro vs Parquet vs CSV"
collection: talks
type: "Do not wait for a schedule to trigger a pipeline"
permalink: /talks/Avro-Parquet-CSV-Gzip
date: 2024-07-01
---

<img width="612" alt="image" src="https://github.com/user-attachments/assets/cbda03bb-977d-4e3a-ad82-546317aedbb5">

I dont want to debate the pros and cons of Parquet and Avro. I will to a performance test.

the [github](https://github.com/nuneskris/FileFormatCompare.git): 

# The data
Using the Parquet Best Practices. We will be using the SalesOrderItems whic has 1930 rows with the below schema.
SALESORDERID: SALESORDERITEM: PRODUCTID: NOTEID: CURRENCY: GROSSAMOUNT:,NETAMOUNT: ,TAXAMOUNT: ,ITEMATPSTATUS: ,OPITEMPOS: ,QUANTITY: ,QUANTITYUNIT: ,DELIVERYDATE: int64

```python
import pandas as pd
import pyarrow as pyar
import pyarrow.parquet as pyarpq
import pyarrow.dataset as pyards
import pyarrow.csv as pyarcsv
```

# Large CSV File Generation
I used combinations of looping throgh files to generate a file with 15,440,000 (fifteen million, four hundred forty thousand) rows.
Size of file: 6.52GB
Type: CSV
Obviously this took a long time to generate.
After compression: 170.MB
Compression ratio: ~40

```python
# Read the CSV file into a DataFrame
csv_file_path = '/Users/krisnunes/Study/python/DataEngineering/DataLakeHouse/FileFormat/Avro/Play/SalesOrdersItems15440000.csv'  # Update with your CSV file path
df = pd.read_csv(csv_file_path)
print(f"Check the original DataFrame:{len(df)}");
# Duplicate the DataFrame
df_duplicate = df.copy()

# Optionally, modify the duplicated DataFrame to differentiate the records
# Example: Add a suffix to the 'NOTEID' field in the duplicated DataFrame
for i in range(1,4):
    print(i)
    df_duplicate['NOTEID'] = df_duplicate['NOTEID'] + f'_dup{i}'
    # Combine the original and duplicated DataFrames
    df_combined = pd.concat([df, df_duplicate], ignore_index=True)
    df_duplicate = df_combined;

# Check the combined DataFrame
print(f"Check the combined DataFrame:{len(df_combined)}");


# Optionally, save the combined DataFrame to a new CSV file
combined_csv_file_path = '/Users/krisnunes/Study/python/DataEngineering/DataLakeHouse/FileFormat/Avro/Play/DataFrame:3860000.csv'  # Update with your desired output CSV file path
df_combined.to_csv(combined_csv_file_path, index=False)

print(f'Combined data saved to {combined_csv_file_path}')

# will use thi dataframe for conversion
df_salesOrder = pd.read_csv('SalesOrdersItems15440000.csv')
```

# Parquet
I built the schema based on a previous demo and reusing it.
SALESORDERID: int64,SALESORDERITEM: int64,PRODUCTID: string,NOTEID: string,CURRENCY: string,GROSSAMOUNT: int64,NETAMOUNT: double,TAXAMOUNT: double,ITEMATPSTATUS: string,OPITEMPOS: string,QUANTITY: int64,QUANTITYUNIT: string,DELIVERYDATE: int64
Size of file: 11.1 MB
Type: Parquet
Time: <1 min
Compression Ratio: 600

```python
# reading the schema from the file
myschema = pyarpq.read_schema("SalesOrderItems.metadata")

# Convert the DataFrame to a PyArrow Table using the schema
sales_order_table = pyar.Table.from_pandas(df_salesOrder, schema=myschema)
pyarpq.write_table(
    sales_order_table,
    'SalesOrderItems_base.parquet'
)
```

## Parquet: Time to read
Time: 40s

```python
df_parquetread = pd.read_parquet('SalesOrderItems_base.parquet') 
print(len(df_parquetread))
```

# Avro: fastavro
Used a similar schema from Parquet. Used fastavro 
Size of file: 6.3 GB
Type: avro
Time: 6 mins
Compression Ratio: 1.0
After compression 178.MB
Compression ratio: ~40

```python
import pandas as pd
import fastavro
import numpy as np

# Define the Avro schema
schema = {
    "type": "record",
    "name": "SalesOrder",
    "fields": [
        {"name": "SALESORDERID", "type": "long"},
        {"name": "SALESORDERITEM", "type": "long"},
        {"name": "PRODUCTID", "type": "string"},
        {"name": "NOTEID", "type": "string"},
        {"name": "CURRENCY", "type": "string"},
        {"name": "GROSSAMOUNT", "type": "long"},
        {"name": "NETAMOUNT", "type": "double"},
        {"name": "TAXAMOUNT", "type": "double"},
        {"name": "ITEMATPSTATUS", "type": "string"},
        {"name": "OPITEMPOS", "type": "string"},
        {"name": "QUANTITY", "type": "long"},
        {"name": "QUANTITYUNIT", "type": "string"},
        {"name": "DELIVERYDATE", "type": "long"}
    ]
}

# Read the CSV file

# Replace nan with an appropriate default value
df = df_salesOrder.replace({np.nan: ""})

# Convert the DataFrame to a list of dictionaries
records = df.to_dict(orient='records')

# Write the Avro file
avro_file = 'output.avro'
with open(avro_file, 'wb') as out:
    fastavro.writer(out, schema, records)

print(f"CSV data has been successfully converted to {avro_file}")
```

## Avro: Time to read
Time: 40s
```python
# Path to the Avro file
avro_file = 'output.avro'

# Read the Avro file into a list of records
records = []

with open(avro_file, 'rb') as file:
    reader = fastavro.reader(file)
    for record in reader:
        records.append(record)
```

# Avro: default avro library
Used a similar schema from Parquet. Used avro 
Size of file: 6.3 GB
Type: avro
Time: 11 mins
Compression Ratio: 1.0

```python
import pandas as pd
import avro.schema
import avro.datafile
import avro.io
import io
import numpy as np


# Define the Avro schema
schema_str = """
{
  "type": "record",
  "name": "SalesOrderItem",
  "fields": [
    {"name": "SALESORDERID", "type": "long"},
    {"name": "SALESORDERITEM", "type": "long"},
    {"name": "PRODUCTID", "type": "string"},
    {"name": "NOTEID", "type": "string"},
    {"name": "CURRENCY", "type": "string"},
    {"name": "GROSSAMOUNT", "type": "long"},
    {"name": "NETAMOUNT", "type": "double"},
    {"name": "TAXAMOUNT", "type": "double"},
    {"name": "ITEMATPSTATUS", "type": "string"},
    {"name": "OPITEMPOS", "type": "string"},
    {"name": "QUANTITY", "type": "long"},
    {"name": "QUANTITYUNIT", "type": "string"},
    {"name": "DELIVERYDATE", "type": "long"}
  ]
}
"""
# Replace nan with an appropriate default value
df_salesOrdersamll = df_salesOrder.replace({np.nan: ""})

print(len(df_salesOrdersamll))

schema = avro.schema.Parse(schema_str)
# Convert the DataFrame to a list of dictionaries
records = df_salesOrdersamll.to_dict(orient='records')

# Write the Avro file
avro_file = 'output.avro'
with open(avro_file, 'wb') as out:
    writer = avro.datafile.DataFileWriter(out, avro.io.DatumWriter(), schema)
    for record in records:
        writer.append(record)
    writer.close()
```

![image](https://github.com/user-attachments/assets/f1b9ec2a-eb9e-46a1-b521-ff4215262fcb)

It is obvious why Parquet performed better. I am sure we can fine tune things. But out of the box, parquet wins a flawless victory.
