---
title: "Staging with Column Transformations"
collection: teaching
type: "DataLake"
permalink: /teaching/DBT-ColumnTransformations
date: 2024-06-01
venue: "Glue"
location: "AWS"
---

I will be Using DBT to run some tranformations on Snowflake. I will be using the ERP data which I have used in multiple demos. 
Data was loaded into Snowflake stages from this [demo](https://nuneskris.github.io/teaching/Snowflake-S3-Integration).

<img width="300" alt="image" src="https://github.com/user-attachments/assets/062203d2-9899-4635-af22-51546adc694f">


# Objectives

#### Column Level Transformations
1. Renaming Columns: Change column names for consistency or clarity.
```
 LOGINNAME as USERNAME
```
2. Column Data Type Conversion: Changing data type
```
{{to_date_number_YYYYMMDD('VALIDITY_STARTDATE') }} as VALIDITY_STARTDATE
```
3. Column Merging: Combine multiple columns into one.
```
CONCAT_WS(' ', NAME_FIRST, NAME_MIDDLE, NAME_LAST, NAME_INITIALS) as full_name
```
4. Handling Missing Values: Fill, drop, or impute missing values
```
coalesce(NAME_FIRST, '') as NAME_FIRST
```
6. Regular Expression Transformations: Use expressions on string values
```
REGEXP_SUBSTR(
	REGEXP_REPLACE
		(  WEBADDRESS, 'https?://|www\.|/$', ''), '^[^/]+') AS EXTRACTED_DOMAIN
```
7. Conditional Transformations: Apply transformations based on conditions.
```
	CASE 
	WHEN ADDRESSTYPE = 1 THEN 'BILL'
        WHEN ADDRESSTYPE = 2 THEN 'SHIP'
        ELSE CAST(ADDRESSTYPE AS STRING)
	END AS ADDRESSTYPE_TRANSFORMED
```
8. String Transformations: Perform operations like trimming, padding, and case conversion.
```
   UPPER(CITY) AS CITY
```
9. Column Filtering: Remove unwanted columns from the dataset.
10. Date and Time Transformations: Extract parts of dates, convert time zones, or format dates.
```
CAST(SUBSTRING(CAST(FISCALYEARPERIOD AS STRING), 1, 4) AS INTEGER) AS FISCALYEAR,
CAST(SUBSTRING(CAST(FISCALYEARPERIOD AS STRING), 5, 3) AS INTEGER) AS FISCALMONTH,
CASE
    WHEN CAST(SUBSTRING(CAST(FISCALYEARPERIOD AS STRING), 5, 3) AS INTEGER) IN (1, 2, 3) THEN 1
    WHEN CAST(SUBSTRING(CAST(FISCALYEARPERIOD AS STRING), 5, 3) AS INTEGER) IN (4, 5, 6) THEN 2
    WHEN CAST(SUBSTRING(CAST(FISCALYEARPERIOD AS STRING), 5, 3) AS INTEGER) IN (7, 8, 9) THEN 3
    ELSE 4
END AS FISCALQUARTER
```
14. Column Splitting: Split a column into multiple columns based on a delimiter. ```SPLIT_PART(EMAILADDRESS, '@', 2) AS EMAILDOMAIN```


#### Other
1. Macros developmennt

# Installation notes
1. The below is what I always use to setup DBT Cloud on Snowflake
* The main configuration is the default schema: ERP_SCHEMA
https://docs.getdbt.com/guides/snowflake?step=4
2. Configure DBT on Github if needed.

# Hello DBT

We will staging data from the src by performing column transformtions on the Employee table This is the inputdata

<img width="300" alt="image" src="/images/teachings/snowflake/SnowflakeFirstLoad.png">

## Using the default package.
The first run will have only one model which will be a src extraction.

<img width="300" alt="image" src="/images/teachings/snowflake/folderDBT.png">

## Using a simple dbt_project.yml
* name: 'dbterp' : Name of the project
* +materialized: view : I am setting this as view so that I am able to look at the outcomes in snowflake.

```yaml
name: 'dbterp'
version: '1.0.0'
config-version: 2

# This setting configures which "profile" dbt uses for this project.
profile: 'dbterp'

# These configurations specify where dbt should look for different types of files.
# The `model-paths` config, for example, states that models in this project can be
# found in the "models/" directory. You probably won't need to change these!
model-paths: ["models"]
analysis-paths: ["analyses"]
test-paths: ["tests"]
seed-paths: ["seeds"]
macro-paths: ["macros"]
snapshot-paths: ["snapshots"]
asset-paths: ["assets"]

target-path: "target"  # directory which will store compiled SQL files
clean-targets:         # directories to be removed by `dbt clean`
  - "target"
  - "dbt_packages"


# Configuring models
# Full documentation: https://docs.getdbt.com/docs/configuring-models

# In this example config, we tell dbt to build all models in the example/ directory
# as tables. These settings can be overridden in the individual model files
# using the `{{ config(...) }}` macro.
models:
  dbterp:
    +materialized: view
    dim:
      +materialized: ephemeral
    src:
      +materialized: view
    fct:
      +materialized: table
```

The First Model which sources and creates the first staging layer which focuses on Column Transformation
* Renaming Columns: LOGINNAME as USERNAME
* Handling Missing Values: Names. coalesce: Replace null values with a specified value (empty string in this case) to prevent nulls from affecting concatenation.
* Column Merging: merging names into a single column
* Column Data Type Conversion: casting interger date types into a date

```sql
-- models/src/erp/employees/prestage_employees.sql
{% set columns = ["NAME_FIRST", "NAME_MIDDLE", "NAME_LAST", "NAME_INITIALS"] %}
-- Renaming Columns: LOGINNAME as USERNAME
-- Handling Missing Values: Names. coalesce: Replace null values with a specified value (empty string in this case) to prevent nulls from affecting concatenation.
-- Column Merging: merging names into a single column
-- Column Data Type Conversion: casting interger date types into a date
{{
  config(
    schema='erp_etl'
  )
}}
WITH PRESTAGE_EMPLOYEES AS ( SELECT
    EMPLOYEEID,
    coalesce(NAME_FIRST, '') as NAME_FIRST,
    coalesce(NAME_MIDDLE, '') as NAME_MIDDLE,
    coalesce(NAME_LAST, '') as NAME_LAST,
    coalesce(NAME_INITIALS, '') as NAME_INITIALS,
    SEX,
    LANGUAGE,
    PHONENUMBER,
    EMAILADDRESS,
    SPLIT_PART(EMAILADDRESS, '@', 2) AS EMAILDOMAIN,
    LOGINNAME,
    ADDRESSID,
    VALIDITY_STARTDATE,
    VALIDITY_ENDDATE
FROM
DB_PRESTAGE.ERP.EMPLOYEES
)
SELECT
    EMPLOYEEID,
    CONCAT_WS(' ', NAME_FIRST, NAME_MIDDLE, NAME_LAST, NAME_INITIALS) as full_name,
    SEX,
    LANGUAGE,
    PHONENUMBER,
    EMAILDOMAIN,
    EMAILADDRESS,
    LOGINNAME as USERNAME,
    ADDRESSID,
    {{to_date_number_YYYYMMDD('VALIDITY_STARTDATE') }} as VALIDITY_STARTDATE,
    {{to_date_number_YYYYMMDD('VALIDITY_ENDDATE') }} as VALIDITY_ENDDATE
    
FROM
PRESTAGE_EMPLOYEES
```

I have additionally used macros for the column transformation: A dbt macro is a reusable piece of code written in Jinja, a templating language. Macros help automate repetitive tasks, making your dbt project more efficient and maintainable. You can think of a macro as a function that you can call with specific arguments to perform a task or generate code dynamically.

```
-- macros/to_date.sql
{% macro to_date_number_YYYYMMDD(column) %}
  TO_DATE(CAST({{ column }} AS STRING), 'YYYYMMDD')
{% endmacro %}
```

# First Run
A view is created in Snowflake when we run the above model.

<img width="300" alt="image" src="/images/teachings/snowflake/FirstDBTRun.png">

# Cleaning up Addresses
* Column Filtering: STREET BUILDING
* Conditional Transformations: ADDRESSTYPE
* String Transformations: CITY

```sql
-- models/src/erp/addresses/prestage_addresses.sql
-- Column Filtering: STREET BUILDING
-- Column Splitting: ADDRESSTYPE
-- String Transformations: CITY
{{
  config(
    schema='erp_etl'
  )
}}
WITH PRESTAGE_ADDRESSES AS ( SELECT
    *,
    CASE 
        WHEN ADDRESSTYPE = 1 THEN 'BILL'
        WHEN ADDRESSTYPE = 2 THEN 'SHIP'
        ELSE CAST(ADDRESSTYPE AS STRING)
    END AS ADDRESSTYPE_TRANSFORMED
FROM
DB_PRESTAGE.ERP.ADDRESSES
)
SELECT
    ADDRESSID,
	UPPER(CITY) AS CITY,
	POSTALCODE,
	STREET,
	BUILDING,
	COUNTRY,
	REGION,
	ADDRESSTYPE_TRANSFORMED as ADDRESSTYPE,
	{{to_date_number_YYYYMMDD('VALIDITY_STARTDATE') }} as VALIDITY_STARTDATE,
    {{to_date_number_YYYYMMDD('VALIDITY_ENDDATE') }} as VALIDITY_ENDDATE,
	LATITUDE FLOAT,
	LONGITUDE FLOAT
FROM
PRESTAGE_ADDRESSES
```
I extracted a FISCALYEARPERIOD which was in the format YYYYMMM into YEAR, MONTH and QUATER
```sql
-- models/src/erp/salesorders/prestage_salesorders.sql
-- Date and Time Transformations: FISCALYEARPERIOD as FISCALYEAR, FISCALMONTH, FISCALQUARTER
{{
  config(
    schema='erp_etl'
  )
}}

WITH PRESTAGE_SALES_ORDERS AS (
    SELECT
        *,
        CAST(SUBSTRING(CAST(FISCALYEARPERIOD AS STRING), 1, 4) AS INTEGER) AS FISCALYEAR,
        CAST(SUBSTRING(CAST(FISCALYEARPERIOD AS STRING), 5, 3) AS INTEGER) AS FISCALMONTH,
        CASE
            WHEN CAST(SUBSTRING(CAST(FISCALYEARPERIOD AS STRING), 5, 3) AS INTEGER) IN (1, 2, 3) THEN 1
            WHEN CAST(SUBSTRING(CAST(FISCALYEARPERIOD AS STRING), 5, 3) AS INTEGER) IN (4, 5, 6) THEN 2
            WHEN CAST(SUBSTRING(CAST(FISCALYEARPERIOD AS STRING), 5, 3) AS INTEGER) IN (7, 8, 9) THEN 3
            ELSE 4
        END AS FISCALQUARTER
    FROM
        DB_PRESTAGE.ERP.SALES_ORDERS
)

SELECT
    SALESORDERID,
    CREATEDBY,
    {{to_date_number_YYYYMMDD('CREATEDAT') }} AS CREATEDAT,
    CHANGEDBY,
    {{to_date_number_YYYYMMDD('CHANGEDAT') }} AS CHANGEDAT,
    FISCALYEAR,
    FISCALMONTH,
    FISCALQUARTER,
    PARTNERID,
    SALESORG,
    CURRENCY,
    GROSSAMOUNT,
    NETAMOUNT,
    TAXAMOUNT,
    LIFECYCLESTATUS,
    BILLINGSTATUS,
    DELIVERYSTATUS
FROM
    PRESTAGE_SALES_ORDERS

```
Performed a Regular Expression Transformation to remove the domain form the 
```sql
-- models/src/erp/salesorderitems/prestage_businesspartners.sql
-- Regular Expression Transformations
{{
  config(
    schema='erp_etl'
  )
}}
WITH PRESTAGE_BUSINESS_PARTNERS AS ( SELECT
*,
  REGEXP_SUBSTR(
            REGEXP_REPLACE(WEBADDRESS, 'https?://|www\.|/$', ''),  -- Remove common URL parts
            '^[^/]+'
        ) AS EXTRACTED_DOMAIN
FROM
DB_PRESTAGE.ERP.BUSINESS_PARTNERS
)
SELECT
    PARTNERID,
	PARTNERROLE,
	EMAILADDRESS,
	PHONENUMBER,
	EXTRACTED_DOMAIN AS WEBADDRESS,
	ADDRESSID,
	COMPANYNAME,
	LEGALFORM,
	CREATEDBY,
	{{to_date_number_YYYYMMDD('CREATEDAT') }} as CREATEDAT,
	CHANGEDBY,
    {{to_date_number_YYYYMMDD('CHANGEDAT') }} as CHANGEDAT,
	CURRENCY
FROM
PRESTAGE_BUSINESS_PARTNERS
```
