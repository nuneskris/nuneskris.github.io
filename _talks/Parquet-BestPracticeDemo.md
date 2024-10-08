---
title: "Parquet: Best practices demonstration"
collection: talks
type: "Talk"
permalink: /talks/Parquet-BestPracticeDemo
date: 2024-02-01
---
A often overlooked feature of Parquet is its support for Interoperability which is key to enterprise data plaforms which serves different tools and systems, facilitating data exchange and integration.
This is my take on Parquet best practices and I have used python-pyarrow to demonstrate them.

# Parquet - a widely adopted standard file format:  Best practices demonstration



Focus is on
1. Use  Appropriate Data Types
    1.1 Date and Time:
    1.2 Avoid Nulls When Possible
    1.3 Numerical Data
2. Leverage Compression and Encoding
    2.1 Dictionary Encoding
    2.2 Run-Length and Delta Encoding
3. Partitioning
4. Nested Data Structures
5. Schema Evolution

```python
import pandas as pd
import pyarrow as pyar
import pyarrow.parquet as pyarpq
import pyarrow.dataset as pyards
import pyarrow.csv as pyarcsv
```

## Cleaning up the initial schema
To save time creating the schema, we can use the schema which pyarrow automatically generates. 
However it would not be ideal and we would need to redesign the schema based on the best practices.
Though Parquet stores the schema along the with the data, Pyarrow uses a seperate Schema object to manage the schama. This schema can also be seperately saved and managed.
Also as a baseline we will save it as a Parquet and also save the schema as .metadata and redesign the schema.
The schema of tables are printed for reference.


```python
# Read a CSV file into a PyArrow Table and infer the schema
def parse_schema(schema_str):
    type_mapping = {
        "int64": pyar.int64(),
        "string": pyar.string(),
        "float64": pyar.float64(),
        "double": pyar.float64(),
        "null": pyar.string(),
        'binary':pyar.binary()
    }
    fields = []
    for line in schema_str.strip().split('\n'):
        name, type_str = line.split(': ')
        pa_type = type_mapping[type_str]
        fields.append(pyar.field(name, pa_type))
    return pyar.schema(fields)
    
def parseTables():
    tables = ['Addresses','BusinessPartners','ProductCategories','ProductCategoryText','Products','ProductTexts','SalesOrderItems','SalesOrders','Employees']
    for tableName in tables:
        tablepyar = pyarcsv.read_csv('Data/'+tableName+'.csv')
        csvSchema = tablepyar.schema.to_string();
        print(f'_____________ {tableName}__________________')
        parquet_scheama = parse_schema(csvSchema)
        print(parquet_scheama.to_string().replace('\n',','))
        convert_options= pyarcsv.ConvertOptions(column_types=parquet_scheama)
        custom_csv_format = pyards.CsvFileFormat(convert_options=convert_options)
        dataset = pyards.dataset('Data/'+tableName+'.csv', format=custom_csv_format)
        # Write to Parquet
        table_parquet_table = dataset.to_table()
        pyarpq.write_metadata(table_parquet_table.schema,'Data/'+tableName+'.metadata')
        pyarpq.write_table(table_parquet_table, 'Data/'+tableName+ '.parquet')

parseTables();

```

    _____________ Addresses__________________
    ADDRESSID: int64,CITY: string,POSTALCODE: string,STREET: string,BUILDING: int64,COUNTRY: string,REGION: string,ADDRESSTYPE: int64,VALIDITY_STARTDATE: int64,VALIDITY_ENDDATE: int64,LATITUDE: double,LONGITUDE: double
    _____________ BusinessPartners__________________
    PARTNERID: int64,PARTNERROLE: int64,EMAILADDRESS: string,PHONENUMBER: int64,FAXNUMBER: string,WEBADDRESS: string,ADDRESSID: int64,COMPANYNAME: string,LEGALFORM: string,CREATEDBY: int64,CREATEDAT: int64,CHANGEDBY: int64,CHANGEDAT: int64,CURRENCY: string
    _____________ ProductCategories__________________
    PRODCATEGORYID: string,CREATEDBY: int64,CREATEDAT: int64
    _____________ ProductCategoryText__________________
    PRODCATEGORYID: string,LANGUAGE: string,SHORT_DESCR: string,MEDIUM_DESCR: string,LONG_DESCR: string
    _____________ Products__________________
    PRODUCTID: string,TYPECODE: string,PRODCATEGORYID: string,CREATEDBY: int64,CREATEDAT: int64,CHANGEDBY: int64,CHANGEDAT: int64,SUPPLIER_PARTNERID: int64,TAXTARIFFCODE: int64,QUANTITYUNIT: string,WEIGHTMEASURE: double,WEIGHTUNIT: string,CURRENCY: string,PRICE: int64,WIDTH: string,DEPTH: string,HEIGHT: string,DIMENSIONUNIT: string,PRODUCTPICURL: string
    _____________ ProductTexts__________________
    PRODUCTID: string,LANGUAGE: string,SHORT_DESCR: string,MEDIUM_DESCR: string,LONG_DESCR: string
    _____________ SalesOrderItems__________________
    SALESORDERID: int64,SALESORDERITEM: int64,PRODUCTID: string,NOTEID: string,CURRENCY: string,GROSSAMOUNT: int64,NETAMOUNT: double,TAXAMOUNT: double,ITEMATPSTATUS: string,OPITEMPOS: string,QUANTITY: int64,QUANTITYUNIT: string,DELIVERYDATE: int64
    _____________ SalesOrders__________________
    SALESORDERID: int64,CREATEDBY: int64,CREATEDAT: int64,CHANGEDBY: int64,CHANGEDAT: int64,FISCVARIANT: string,FISCALYEARPERIOD: int64,NOTEID: string,PARTNERID: int64,SALESORG: string,CURRENCY: string,GROSSAMOUNT: int64,NETAMOUNT: double,TAXAMOUNT: double,LIFECYCLESTATUS: string,BILLINGSTATUS: string,DELIVERYSTATUS: string
    _____________ Employees__________________
    EMPLOYEEID: int64,NAME_FIRST: string,NAME_MIDDLE: string,NAME_LAST: string,NAME_INITIALS: string,SEX: string,LANGUAGE: string,PHONENUMBER: string,EMAILADDRESS: string,LOGINNAME: string,ADDRESSID: int64,VALIDITY_STARTDATE: int64,VALIDITY_ENDDATE: int64

    
![image](https://github.com/nuneskris/nuneskris.github.io/assets/82786764/7a730e9f-6946-4a86-ab2d-552b28876875)




We will be focusing on SalesOrders and slowly improve on the schema to demonstrate the first few best practices.


```python
df_salesOrder = pd.read_csv('Data/SalesOrders.csv')
schemaString = pyarpq.read_schema("Data/SalesOrders.metadata").to_string().replace('\n',',')
print(f"schema as Sales Order Data  {schemaString}")
df_salesOrder.head(2)
```

    schema as Sales Order Data  SALESORDERID: int64,CREATEDBY: int64,CREATEDAT: int64,CHANGEDBY: int64,CHANGEDAT: int64,FISCVARIANT: string,FISCALYEARPERIOD: int64,NOTEID: string,PARTNERID: int64,SALESORG: string,CURRENCY: string,GROSSAMOUNT: int64,NETAMOUNT: double,TAXAMOUNT: double,LIFECYCLESTATUS: string,BILLINGSTATUS: string,DELIVERYSTATUS: string





<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>SALESORDERID</th>
      <th>CREATEDBY</th>
      <th>CREATEDAT</th>
      <th>CHANGEDBY</th>
      <th>CHANGEDAT</th>
      <th>FISCVARIANT</th>
      <th>FISCALYEARPERIOD</th>
      <th>NOTEID</th>
      <th>PARTNERID</th>
      <th>SALESORG</th>
      <th>CURRENCY</th>
      <th>GROSSAMOUNT</th>
      <th>NETAMOUNT</th>
      <th>TAXAMOUNT</th>
      <th>LIFECYCLESTATUS</th>
      <th>BILLINGSTATUS</th>
      <th>DELIVERYSTATUS</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>500000000</td>
      <td>4</td>
      <td>20180111</td>
      <td>4</td>
      <td>20180116</td>
      <td>K4</td>
      <td>2018001</td>
      <td>NaN</td>
      <td>100000022</td>
      <td>APJ</td>
      <td>USD</td>
      <td>13587</td>
      <td>11888.625</td>
      <td>1698.375</td>
      <td>C</td>
      <td>C</td>
      <td>C</td>
    </tr>
    <tr>
      <th>1</th>
      <td>500000001</td>
      <td>2</td>
      <td>20180112</td>
      <td>2</td>
      <td>20180115</td>
      <td>K4</td>
      <td>2018001</td>
      <td>NaN</td>
      <td>100000026</td>
      <td>EMEA</td>
      <td>USD</td>
      <td>12622</td>
      <td>11044.250</td>
      <td>1577.750</td>
      <td>C</td>
      <td>C</td>
      <td>C</td>
    </tr>
  </tbody>
</table>
</div>



## Best Practice 1. Use  Appropriate Data Types
Choose the Right Data Types for performance and storage
### 1.1 Date and Time: 
Use timestamp or date types for temporal data. The Sales Order has CREATEDAT and CHANGEDAT as int and we wull update it as date32.

We will use the existing metadata schema file for sales order update it and use it to save the modified dataframe.

Step 1: # using pandas to change the string yymmdd into a date 
Step 2: # get the index and update the schema to date32
Step 3: # Convert the DataFrame to a PyArrow Table using the updated schema and save the new parquet file.


```python
file = pyarpq.ParquetFile('Data/SalesOrders.parquet')
print(f"There is a schema which is embedded within the parquet file \n {file.schema}");
```

    There is a schema which is embedded within the parquet file 
     <pyarrow._parquet.ParquetSchema object at 0x11a366d80>
    required group field_id=-1 schema {
      optional int64 field_id=-1 SALESORDERID;
      optional int64 field_id=-1 CREATEDBY;
      optional int64 field_id=-1 CREATEDAT;
      optional int64 field_id=-1 CHANGEDBY;
      optional int64 field_id=-1 CHANGEDAT;
      optional binary field_id=-1 FISCVARIANT (String);
      optional int64 field_id=-1 FISCALYEARPERIOD;
      optional binary field_id=-1 NOTEID (String);
      optional int64 field_id=-1 PARTNERID;
      optional binary field_id=-1 SALESORG (String);
      optional binary field_id=-1 CURRENCY (String);
      optional int64 field_id=-1 GROSSAMOUNT;
      optional double field_id=-1 NETAMOUNT;
      optional double field_id=-1 TAXAMOUNT;
      optional binary field_id=-1 LIFECYCLESTATUS (String);
      optional binary field_id=-1 BILLINGSTATUS (String);
      optional binary field_id=-1 DELIVERYSTATUS (String);
    }
    



```python
# using pandas to change the string yymmdd into a date
df_salesOrder['CREATEDAT'] = pd.to_datetime(df_salesOrder['CREATEDAT'], format='%Y%m%d')
df_salesOrder['CHANGEDAT'] = pd.to_datetime(df_salesOrder['CHANGEDAT'], format='%Y%m%d')

# reading the schema from the file
myschema = pyarpq.read_schema("Data/SalesOrders.metadata")
# get the index and update the schema to date32. date64 is for ms which is not required
index = pyar.Schema.get_field_index(myschema, 'CREATEDAT')
myschema = pyar.Schema.set(myschema, index, pyar.field('CREATEDAT', pyar.date32()))
index = pyar.Schema.get_field_index(myschema, 'CHANGEDAT')
myschema = pyar.Schema.set(myschema, index, pyar.field('CHANGEDAT', pyar.date32()))

# Convert the DataFrame to a PyArrow Table using the schema
sales_order_table = pyar.Table.from_pandas(df_salesOrder, schema=myschema)
pyarpq.write_table(
    sales_order_table,
    'Data/SalesOrders.parquet'
)
pyarpq.write_metadata(myschema,'Data/SalesOrders.metadata')
# printing the metadata so we are replacing newlines to read it easily
updatedSchema = myschema.to_string().replace('\n',',')

print(f"the new updated schema --> CREATEDAT and CHANGEDAT are updated to date32 format ")
print('----------------------------------------------------------------------------------------------------------')
print(updatedSchema)
updatedParquetfile = pyarpq.ParquetFile('Data/SalesOrders.parquet')
print('----------------------------------------------------------------------------------------------------------')
print('Even the schema on attached to the parquet file is updated.')
print('----------------------------------------------------------------------------------------------------------')
updatedParquetfile.schema
```

    the new updated schema --> CREATEDAT and CHANGEDAT are updated to date32 format 
    ----------------------------------------------------------------------------------------------------------
    SALESORDERID: int64,CREATEDBY: int64,CREATEDAT: date32[day],CHANGEDBY: int64,CHANGEDAT: date32[day],FISCVARIANT: string,FISCALYEARPERIOD: int64,NOTEID: string,PARTNERID: int64,SALESORG: string,CURRENCY: string,GROSSAMOUNT: int64,NETAMOUNT: double,TAXAMOUNT: double,LIFECYCLESTATUS: string,BILLINGSTATUS: string,DELIVERYSTATUS: string
    ----------------------------------------------------------------------------------------------------------
    Even the schema on attached to the parquet file is updated.
    ----------------------------------------------------------------------------------------------------------





    <pyarrow._parquet.ParquetSchema object at 0x118e5cec0>
    required group field_id=-1 schema {
      optional int64 field_id=-1 SALESORDERID;
      optional int64 field_id=-1 CREATEDBY;
      optional int32 field_id=-1 CREATEDAT (Date);
      optional int64 field_id=-1 CHANGEDBY;
      optional int32 field_id=-1 CHANGEDAT (Date);
      optional binary field_id=-1 FISCVARIANT (String);
      optional int64 field_id=-1 FISCALYEARPERIOD;
      optional binary field_id=-1 NOTEID (String);
      optional int64 field_id=-1 PARTNERID;
      optional binary field_id=-1 SALESORG (String);
      optional binary field_id=-1 CURRENCY (String);
      optional int64 field_id=-1 GROSSAMOUNT;
      optional double field_id=-1 NETAMOUNT;
      optional double field_id=-1 TAXAMOUNT;
      optional binary field_id=-1 LIFECYCLESTATUS (String);
      optional binary field_id=-1 BILLINGSTATUS (String);
      optional binary field_id=-1 DELIVERYSTATUS (String);
    }



### 1.2 Avoid Nulls When Possible: 
Design the schema to minimize the use of nulls, as they can affect performance.
NOTEID is null and we will drop the column from the dataframe and also update the schema


```python
# defining this as a function as we will use this a few times.
def updateParquetAndMetaData(df_salesOrder, myschema):
    sales_order_table = pyar.Table.from_pandas(df_salesOrder, schema=myschema)
    pyarpq.write_table(
        sales_order_table,
        'Data/SalesOrders.parquet'
    )
    pyarpq.write_metadata(myschema,'Data/SalesOrders.metadata')
    updatedSchema = myschema.to_string().replace('\n',',')
    print(updatedSchema)
    
# Using Pandas to drop the column 
df_salesOrder = df_salesOrder.drop(['NOTEID'],axis =1)
# removing the column from the meta data too.
index = pyar.Schema.get_field_index(myschema, 'NOTEID')
myschema = pyar.Schema.remove(myschema, index)

print("the new updated schema --> removed the NOTEID \n")
print('----------------------------------------------------------------------------------------------------------')
updateParquetAndMetaData(df_salesOrder, myschema)

```

    the new updated schema --> removed the NOTEID 
    
    ----------------------------------------------------------------------------------------------------------
    SALESORDERID: int64,CREATEDBY: int64,CREATEDAT: date32[day],CHANGEDBY: int64,CHANGEDAT: date32[day],FISCVARIANT: string,FISCALYEARPERIOD: int64,PARTNERID: int64,SALESORG: string,CURRENCY: string,GROSSAMOUNT: int64,NETAMOUNT: double,TAXAMOUNT: double,LIFECYCLESTATUS: string,BILLINGSTATUS: string,DELIVERYSTATUS: string


### 1.3 Numerical Data: Use integer or floating-point types to make them more appropriate.
Modifying float64 to float32 as it would suffice for the values we would need.


```python
index = pyar.Schema.get_field_index(myschema, 'TAXAMOUNT')
myschema = pyar.Schema.set(myschema, index, pyar.field('TAXAMOUNT', pyar.float32()))
index = pyar.Schema.get_field_index(myschema, 'NETAMOUNT')
myschema = pyar.Schema.set(myschema, index, pyar.field('NETAMOUNT', pyar.float32()))
index = pyar.Schema.get_field_index(myschema, 'GROSSAMOUNT')
myschema = pyar.Schema.set(myschema, index, pyar.field('GROSSAMOUNT', pyar.int32()))

print(f"the new updated schema -> abover 3 columns would be updated to float from double")
print('----------------------------------------------------------------------------------------------------------')
updateParquetAndMetaData(df_salesOrder, myschema)
```

    the new updated schema -> abover 3 columns would be updated to float from double
    ----------------------------------------------------------------------------------------------------------
    SALESORDERID: int64,CREATEDBY: int64,CREATEDAT: date32[day],CHANGEDBY: int64,CHANGEDAT: date32[day],FISCVARIANT: string,FISCALYEARPERIOD: int64,PARTNERID: int64,SALESORG: string,CURRENCY: string,GROSSAMOUNT: int32,NETAMOUNT: float,TAXAMOUNT: float,LIFECYCLESTATUS: string,BILLINGSTATUS: string,DELIVERYSTATUS: string


## Best Practice 2. Leverage Compression and Encoding
Column-Specific Compression: Choose appropriate compression methods for each column based on the data type and characteristics.
Encoding Techniques: Use encoding techniques like dictionary encoding, run-length encoding, or delta encoding to further optimize storage and performance.
Text Data: Use string types, but avoid using strings for numerical or categorical data.

### 2.1 Dictionary Encoding
dictionary<values=string, indices=int8, ordered=0>: This indicates that the category column is stored as a dictionary-encoded column, where the unique values are of type string, and the indices pointing to these values are of type int8.
Benefits of Dictionary Encoding
Storage Efficiency: Only unique values are stored once, reducing storage space.
Query Performance: Lookups are faster as the data is compact and indices are used for comparison.
#### When to use??? Use for columns with low cardinality and high repetition (use_dictionary parameter).

Step 1: Dataframe.astype('category') will automatically convert to catagory 
Step 2: pyar.field('LIFECYCLESTATUS', pyar.dictionary(pyar.int8(), pyar.utf8())) to define the category



```python
# Convert the 'category' column to a categorical type
df_salesOrder['LIFECYCLESTATUS'] = df_salesOrder['LIFECYCLESTATUS'].astype('category')
df_salesOrder['BILLINGSTATUS'] = df_salesOrder['BILLINGSTATUS'].astype('category')
df_salesOrder['DELIVERYSTATUS'] = df_salesOrder['DELIVERYSTATUS'].astype('category')
df_salesOrder['SALESORG'] = df_salesOrder['SALESORG'].astype('category')

index = pyar.Schema.get_field_index(myschema, 'LIFECYCLESTATUS')
myschema = pyar.Schema.set(myschema, index, pyar.field('LIFECYCLESTATUS', pyar.dictionary(pyar.int8(), pyar.utf8())))
index = pyar.Schema.get_field_index(myschema, 'BILLINGSTATUS')
myschema = pyar.Schema.set(myschema, index, pyar.field('BILLINGSTATUS', pyar.dictionary(pyar.int8(), pyar.utf8())))
index = pyar.Schema.get_field_index(myschema, 'DELIVERYSTATUS')
myschema = pyar.Schema.set(myschema, index, pyar.field('DELIVERYSTATUS', pyar.dictionary(pyar.int8(), pyar.utf8())))
index = pyar.Schema.get_field_index(myschema, 'SALESORG')
myschema = pyar.Schema.set(myschema, index, pyar.field('SALESORG', pyar.dictionary(pyar.int8(), pyar.utf8())))

print(f"We can see the columns are updated to a dictionary")
print('----------------------------------------------------------------------------------------------------------')
updateParquetAndMetaData(df_salesOrder, myschema)
```

    We can see the columns are updated to a dictionary
    ----------------------------------------------------------------------------------------------------------
    SALESORDERID: int64,CREATEDBY: int64,CREATEDAT: date32[day],CHANGEDBY: int64,CHANGEDAT: date32[day],FISCVARIANT: string,FISCALYEARPERIOD: int64,PARTNERID: int64,SALESORG: dictionary<values=string, indices=int8, ordered=0>,CURRENCY: string,GROSSAMOUNT: int32,NETAMOUNT: float,TAXAMOUNT: float,LIFECYCLESTATUS: dictionary<values=string, indices=int8, ordered=0>,BILLINGSTATUS: dictionary<values=string, indices=int8, ordered=0>,DELIVERYSTATUS: dictionary<values=string, indices=int8, ordered=0>


### 2.2 Run-Length and Delta Encoding 
RLE is a simple and efficient form of data compression where sequences of the same data value (runs) are stored as a single data value and a count, rather than as the original run. 
This technique is particularly effective for data that contains many consecutive repeated values.
Default encoding is typically enabled which can include RLE for certain data types. However if we want to maintain Dictionary encoding for specific colummns we would need to specify them.
PyArrow automatically applies delta encoding to integer columns where appropriate

Step: # Write the table to a Parquet file with dictionary encoding for 'category' and default encoding for others



```python
# Write the table to a Parquet file with dictionary encoding for 'category' and default encoding for others
pyarpq.write_table(
    pyar.Table.from_pandas(df_salesOrder, preserve_index=False),
    'Data/SalesOrders.parquet',
    use_dictionary=['SALESORG','LIFECYCLESTATUS','BILLINGSTATUS','DELIVERYSTATUS'],  # Specify dictionary encoding for the 'category' column
    compression='SNAPPY',  # Use compression for better storage efficiency
    data_page_version='2.0',  # Use data page version 2.0 for better performance
    write_statistics=True
)
```

## Best Practice 3  Partitioning
Partitioning Strategy: Design your schema to include partition columns that make sense for your queries. Common partition columns are dates, geographic regions, or user IDs.

Balance Partition Size: Ensure that partitions are neither too large nor too small. Aim for a partition size that strikes a balance between query performance and manageability.

We will split fiscal year which is a yyyy0mm format and partition it year and month. This is very typical.

Step 1: spliting the FISCALYEARPERIOD into Month and year
Step 2: Now we are going to update the schema with Fiscalyear and Fiscal month.
Step 3: Generate Table from DF
Step 4: write_to_dataset and mention and mention partitions.



```python
# spliting the FISCALYEARPERIOD into Month and year
# Also we will remove the FISCALYEARPERIOD column
df_salesOrder[['FISCALYEAR', 'FISCALMONTH']] = df_salesOrder['FISCALYEARPERIOD'].apply(lambda x: pd.Series([int(str(x)[:4]), int(str(x)[5:])])  )
# Now we are going to update the schema with Fiscalyear and Fiscal month. They will not be columns but folders
new_fields =[ pyar.field('FISCALYEAR',pyar.int64()),
                                    pyar.field('FISCALMONTH',pyar.int64())
                                    ]
# Combine existing fields with new fields
myschema = pyar.schema(list(myschema) + new_fields)
df_salesOrder = df_salesOrder.drop(['FISCALYEARPERIOD'], axis = 1);
# removing the column from the meta data too.
index = pyar.Schema.get_field_index(myschema, 'FISCALYEARPERIOD')
myschema = pyar.Schema.remove(myschema, index)
# Generate Table from DF
table_SalesOrder = pyar.Table.from_pandas(df_salesOrder, myschema )
partition_cols = ['FISCALYEAR', 'FISCALMONTH']
pyarpq.write_to_dataset(
    table_SalesOrder,
    root_path='SalerOrderPartition',
    use_dictionary=['SALESORG','LIFECYCLESTATUS','BILLINGSTATUS','DELIVERYSTATUS'],  # Specify dictionary encoding for the 'category' column
    compression='SNAPPY',  # Use compression for better storage efficiency
    partition_cols= partition_cols,
    schema = myschema
)
pyarpq.write_metadata(myschema,'Data/SalesOrders.metadata')
# print schema
updatedSchema = myschema.to_string().replace('\n',',')
print(f"Updated Schema \n {updatedSchema}")
df_salesOrder.head(3)
```

    Updated Schema 
     SALESORDERID: int64,CREATEDBY: int64,CREATEDAT: date32[day],CHANGEDBY: int64,CHANGEDAT: date32[day],FISCVARIANT: string,PARTNERID: int64,SALESORG: dictionary<values=string, indices=int8, ordered=0>,CURRENCY: string,GROSSAMOUNT: int32,NETAMOUNT: float,TAXAMOUNT: float,LIFECYCLESTATUS: dictionary<values=string, indices=int8, ordered=0>,BILLINGSTATUS: dictionary<values=string, indices=int8, ordered=0>,DELIVERYSTATUS: dictionary<values=string, indices=int8, ordered=0>,FISCALYEAR: int64,FISCALMONTH: int64





<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>SALESORDERID</th>
      <th>CREATEDBY</th>
      <th>CREATEDAT</th>
      <th>CHANGEDBY</th>
      <th>CHANGEDAT</th>
      <th>FISCVARIANT</th>
      <th>PARTNERID</th>
      <th>SALESORG</th>
      <th>CURRENCY</th>
      <th>GROSSAMOUNT</th>
      <th>NETAMOUNT</th>
      <th>TAXAMOUNT</th>
      <th>LIFECYCLESTATUS</th>
      <th>BILLINGSTATUS</th>
      <th>DELIVERYSTATUS</th>
      <th>FISCALYEAR</th>
      <th>FISCALMONTH</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>500000000</td>
      <td>4</td>
      <td>2018-01-11</td>
      <td>4</td>
      <td>2018-01-16</td>
      <td>K4</td>
      <td>100000022</td>
      <td>APJ</td>
      <td>USD</td>
      <td>13587</td>
      <td>11888.625</td>
      <td>1698.375</td>
      <td>C</td>
      <td>C</td>
      <td>C</td>
      <td>2018</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>500000001</td>
      <td>2</td>
      <td>2018-01-12</td>
      <td>2</td>
      <td>2018-01-15</td>
      <td>K4</td>
      <td>100000026</td>
      <td>EMEA</td>
      <td>USD</td>
      <td>12622</td>
      <td>11044.250</td>
      <td>1577.750</td>
      <td>C</td>
      <td>C</td>
      <td>C</td>
      <td>2018</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>500000002</td>
      <td>5</td>
      <td>2018-01-15</td>
      <td>5</td>
      <td>2018-01-20</td>
      <td>K4</td>
      <td>100000018</td>
      <td>APJ</td>
      <td>USD</td>
      <td>45655</td>
      <td>39948.125</td>
      <td>5706.875</td>
      <td>C</td>
      <td>C</td>
      <td>C</td>
      <td>2018</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>

![image](https://github.com/nuneskris/nuneskris.github.io/assets/82786764/e248b65d-d0e1-49d8-9a31-5b2669cd1c23)



## Best Practice 4  Nested Data Structures
Use Nested Structures: Parquet supports complex nested data structures (e.g., maps, arrays, structs). Use them to represent hierarchical data more efficiently.
Flatten When Necessary: If nested structures complicate querying, consider flattening the schema.
##### Normalized Data Makes Life Easy
Avoid excessive nesting. Deeply nested structures can be harder to query and may impact performance. Normalize your data where possible.
##### When to Nest Data: 
1. When columns are always accessed together, nesting can make sense.
2. Read-heavy where performance if impacted because of the need for joins.
3. Self-Contained Data
##### When to Normalize:
1. For nested structures, use struct, list, or map types where appropriate.
1. If nesting leads to significant data redundancy (e.g., repeating user information for every order)
2. When down stream systems need to process data. ETL pipelines work better with normalized tables
3. When nested data have relationships with other data entities which are independently managed or queried.

Reduce Redundancy: Avoid storing redundant data. 
Normalize your schema to reduce duplication, but balance it with the need for efficient querying.

To demonstrate Nesting we will Nest the Business partners and Address. Another typical usecase

Step 1: To make this convenient, I set a prefix on these columns and so that they can be easily identified and then removed once the df is merged.
Step 2: Merging the dataframe
Step 3: create Dictionary for the nested df (format required for the nesting)
Step 4: Drop the of the uneeed columns
Step 5: Update new schema with struct data type
Step 6: generate parquet


```python
df_BusinessPartners = pd.read_csv('Data/BusinessPartners.csv')
df_Addresses = pd.read_csv('Data/Addresses.csv')
# creating a prefix
for col in df_Addresses.columns:
    df_Addresses.rename(columns = {col:'Addresses_'+col}, inplace = True)
df_Addresses['ADDRESSID'] = df_Addresses['Addresses_ADDRESSID'].copy()
# Merging the dataframe
merged_df = pd.merge(df_BusinessPartners, df_Addresses, on='ADDRESSID')
```


```python

def nestColumn(X):
    nested = {};
    for col in df_Addresses.columns:
        if('Addresses_' in col ):
            nested.update({col.replace('Addresses_',''): X[col]})
    return nested
# create Dictionary for the nested df (format required for the nesting)
merged_df['ADDRESS'] = merged_df.apply(
    lambda row: 
        nestColumn(row)
    ,
    axis=1
)
for col in merged_df.columns:
    if('Addresses_' in col ):
        merged_df = merged_df.drop(columns = [col], axis= 1)

merged_df = merged_df.drop(columns='ADDRESSID', axis = 1)

merged_df.head(2)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>PARTNERID</th>
      <th>PARTNERROLE</th>
      <th>EMAILADDRESS</th>
      <th>PHONENUMBER</th>
      <th>FAXNUMBER</th>
      <th>WEBADDRESS</th>
      <th>COMPANYNAME</th>
      <th>LEGALFORM</th>
      <th>CREATEDBY</th>
      <th>CREATEDAT</th>
      <th>CHANGEDBY</th>
      <th>CHANGEDAT</th>
      <th>CURRENCY</th>
      <th>ADDRESS</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>100000000</td>
      <td>2</td>
      <td>maria.brown@all4bikes.com</td>
      <td>622734567</td>
      <td>NaN</td>
      <td>http://www.all4bikes.com</td>
      <td>All For Bikes</td>
      <td>Inc.</td>
      <td>10</td>
      <td>20181003</td>
      <td>10</td>
      <td>20181003</td>
      <td>USD</td>
      <td>{'ADDRESSID': 1000000034, 'CITY': 'West Nyack'...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>100000001</td>
      <td>2</td>
      <td>bob.buyer@amazebikes.com</td>
      <td>3088530</td>
      <td>NaN</td>
      <td>http://www.amazebikes.com</td>
      <td>Amaze Bikes Inc</td>
      <td>Inc.</td>
      <td>13</td>
      <td>20181003</td>
      <td>13</td>
      <td>20181003</td>
      <td>USD</td>
      <td>{'ADDRESSID': 1000000035, 'CITY': 'Fair Oaks',...</td>
    </tr>
  </tbody>
</table>
</div>




```python

# Paths to the .metadata files
metadata_pathbp = 'Data/BusinessPartners.metadata'
metadata_patha = 'Data/Addresses.metadata'

# Read the metadata
metadatabp = pyarpq.read_metadata(metadata_pathbp)
metadataa = pyarpq.read_metadata(metadata_patha)

# Extract the schemas from the metadata
schemaso = metadatabp.schema.to_arrow_schema()
schemasoI = metadataa.schema.to_arrow_schema()

new_fields =[ pyar.field('ADDRESS',pyar.struct(list(schemasoI))) ]
# Combine existing fields with new fields
myschema_businessPartners = pyar.schema(list(schemaso) + new_fields)
# removing the column from the meta data too.
index = pyar.Schema.get_field_index(myschema_businessPartners, 'ADDRESSID')
myschema_businessPartners = pyar.Schema.remove(myschema_businessPartners, index)
print(f"Updated Schema \n ----------------------------------------------------------------- \n{myschema_businessPartners}")

```

    Updated Schema 
     ----------------------------------------------------------------- 
    PARTNERID: int64
    PARTNERROLE: int64
    EMAILADDRESS: string
    PHONENUMBER: int64
    FAXNUMBER: string
    WEBADDRESS: string
    COMPANYNAME: string
    LEGALFORM: string
    CREATEDBY: int64
    CREATEDAT: int64
    CHANGEDBY: int64
    CHANGEDAT: int64
    CURRENCY: string
    ADDRESS: struct<ADDRESSID: int64, CITY: string, POSTALCODE: string, STREET: string, BUILDING: int64, COUNTRY: string, REGION: string, ADDRESSTYPE: int64, VALIDITY_STARTDATE: int64, VALIDITY_ENDDATE: int64, LATITUDE: double, LONGITUDE: double>
      child 0, ADDRESSID: int64
      child 1, CITY: string
      child 2, POSTALCODE: string
      child 3, STREET: string
      child 4, BUILDING: int64
      child 5, COUNTRY: string
      child 6, REGION: string
      child 7, ADDRESSTYPE: int64
      child 8, VALIDITY_STARTDATE: int64
      child 9, VALIDITY_ENDDATE: int64
      child 10, LATITUDE: double
      child 11, LONGITUDE: double



```python
table_BusinessPartners= pyar.Table.from_pandas(merged_df, myschema_businessPartners )
pyarpq.write_table(
    table_BusinessPartners,
    'Data/BusinessPartners.parquet'
)
pyarpq.write_metadata(myschema_businessPartners,'Data/BusinessPartners.metadata')
```

# ## Best Practice 5 Schema Evolution 
Plan for Changes: Design your schema to be flexible for future changes. Parquet supports schema evolution, allowing you to add or modify columns without breaking existing queries.
Backward and Forward Compatibility: Ensure that changes to the schema maintain compatibility with existing data and applications.

Schema evolution in Parquet refers to the capability of the Parquet file format to handle changes in the schema (structure) of the data over time without breaking compatibility with older data. This feature allows you to modify the schema, such as adding new columns or changing data types, while still being able to read and write data using the updated schema.

Key Aspects of Schema Evolution in Parquet
Adding Columns: You can add new columns to the schema. When reading data, the new columns will have null or default values for the old data that does not contain these columns.

Removing Columns: You can remove columns from the schema. When reading older data, the removed columns will be ignored.

Changing Data Types: Parquet supports certain types of data type changes (e.g., widening types like int to long). However, not all type changes are supported, and some may require more careful handling or data migration.

Reordering Columns: Parquet does not require columns to be in a specific order, so reordering columns in the schema does not affect data compatibility.

Benefits of Schema Evolution
Flexibility: Allows the schema to evolve as the data requirements change over time without needing to rewrite all the historical data.
Backward Compatibility: Ensures that older data can still be read with new schema versions.
Forward Compatibility: Ensures that new data written with an updated schema can still be read by systems expecting the old schema.


```python
schema_ProductCategories = pyarpq.read_metadata('Data/ProductCategories.metadata').schema.to_arrow_schema()
print(f"Schema for ProductCategories:\n{schema_ProductCategories}")
df_ProductCategories = pd.read_parquet('Data/ProductCategories.parquet') 
df_ProductCategories
```

    Schema for ProductCategories:
    PRODCATEGORYID: string
    CREATEDBY: int64
    CREATEDAT: int64





<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>PRODCATEGORYID</th>
      <th>CREATEDBY</th>
      <th>CREATEDAT</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>RO</td>
      <td>12</td>
      <td>20181003</td>
    </tr>
    <tr>
      <th>1</th>
      <td>BX</td>
      <td>4</td>
      <td>20181003</td>
    </tr>
    <tr>
      <th>2</th>
      <td>CC</td>
      <td>7</td>
      <td>20181003</td>
    </tr>
    <tr>
      <th>3</th>
      <td>MB</td>
      <td>11</td>
      <td>20181003</td>
    </tr>
    <tr>
      <th>4</th>
      <td>RC</td>
      <td>9</td>
      <td>20181003</td>
    </tr>
    <tr>
      <th>5</th>
      <td>DB</td>
      <td>8</td>
      <td>20181003</td>
    </tr>
    <tr>
      <th>6</th>
      <td>EB</td>
      <td>11</td>
      <td>20181003</td>
    </tr>
    <tr>
      <th>7</th>
      <td>CB</td>
      <td>10</td>
      <td>20181003</td>
    </tr>
    <tr>
      <th>8</th>
      <td>HB</td>
      <td>6</td>
      <td>20181003</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Updated DataFrame with new column 'PRODUCTMARKET'
df_updated = pd.DataFrame({
    'PRODCATEGORYID': ['BO', 'GA'],
    'CREATEDBY': [2, 3],
    'CREATEDAT': [20191003, 20191003],
    'PRODUCTMARKET': ['B2B', 'B2C'],
})

new_fields =[ pyar.field('PRODUCTMARKET',pyar.string()) ]
# Combine existing fields with new fields
schema_ProductCategories = pyar.schema(list(schema_ProductCategories) + new_fields)

# Convert to PyArrow Table and write to Parquet with updated schema
tableUpdated = pyar.Table.from_pandas(df_updated, schema_ProductCategories)
# Read the original Parquet file
original_table = pyarpq.read_table('Data/ProductCategories.parquet')
# Concatenate the original and updated tables

combined_table = pyar.concat_tables([original_table, tableUpdated],  promote_options="default")
pyarpq.write_table(combined_table, 'Data/ProductCategories.parquet')

```


```python
df_ProductCategories = pd.read_parquet('Data/ProductCategories.parquet') 
df_ProductCategories
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>PRODCATEGORYID</th>
      <th>CREATEDBY</th>
      <th>CREATEDAT</th>
      <th>PRODUCTMARKET</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>RO</td>
      <td>12</td>
      <td>20181003</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1</th>
      <td>BX</td>
      <td>4</td>
      <td>20181003</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2</th>
      <td>CC</td>
      <td>7</td>
      <td>20181003</td>
      <td>None</td>
    </tr>
    <tr>
      <th>3</th>
      <td>MB</td>
      <td>11</td>
      <td>20181003</td>
      <td>None</td>
    </tr>
    <tr>
      <th>4</th>
      <td>RC</td>
      <td>9</td>
      <td>20181003</td>
      <td>None</td>
    </tr>
    <tr>
      <th>5</th>
      <td>DB</td>
      <td>8</td>
      <td>20181003</td>
      <td>None</td>
    </tr>
    <tr>
      <th>6</th>
      <td>EB</td>
      <td>11</td>
      <td>20181003</td>
      <td>None</td>
    </tr>
    <tr>
      <th>7</th>
      <td>CB</td>
      <td>10</td>
      <td>20181003</td>
      <td>None</td>
    </tr>
    <tr>
      <th>8</th>
      <td>HB</td>
      <td>6</td>
      <td>20181003</td>
      <td>None</td>
    </tr>
    <tr>
      <th>9</th>
      <td>BO</td>
      <td>2</td>
      <td>20191003</td>
      <td>B2B</td>
    </tr>
    <tr>
      <th>10</th>
      <td>GA</td>
      <td>3</td>
      <td>20191003</td>
      <td>B2C</td>
    </tr>
  </tbody>
</table>
</div>





