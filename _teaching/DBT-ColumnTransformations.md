---
title: "DBT-ColumnTransformations"
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
1. Renaming Columns: Change column names for consistency or clarity. ``` LOGINNAME as USERNAME```
2. Column Data Type Conversion: Changing data type ```{{to_date_number_YYYYMMDD('VALIDITY_STARTDATE') }} as VALIDITY_STARTDATE```
3. Column Merging: Combine multiple columns into one. ``` CONCAT_WS(' ', NAME_FIRST, NAME_MIDDLE, NAME_LAST, NAME_INITIALS) as full_name,```
4. Handling Missing Values: Fill, drop, or impute missing values : ``` coalesce(NAME_FIRST, '') as NAME_FIRST```

#### Other
1. Macros developmennt
2. 

# Installation notes
1. The below is what I always use to setup DBT Cloud on Snowflake
* The main configuration is the default schema: ERP_SCHEMA
https://docs.getdbt.com/guides/snowflake?step=4
2. Configure DBT on Github if needed.

# Hello DBT

We will staging data from the src by performing column transformtions on the Employee table This is the inputdata

![image](https://github.com/user-attachments/assets/c5db2e55-1735-411a-9557-0b2c2bf6a1f7)

## Using the default package.
The first run will have only one model which will be a src extraction.

![image](https://github.com/user-attachments/assets/396a8156-afc0-4bb0-840f-0e04380ec24e)

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


```sql
-- models/src/erp/employees/prestage_employees.sql
{% set columns = ["NAME_FIRST", "NAME_MIDDLE", "NAME_LAST", "NAME_INITIALS"] %}
-- coalesce: Replace null values with a specified value (empty string in this case) to prevent nulls from affecting concatenation.
-- merging names into a single column
-- casting interger date types into a date
-- Renaming Columns: 
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
    EMAILADDRESS,
    LOGINNAME as USERNAME,
    ADDRESSID,
    {{to_date_number_YYYYMMDD('VALIDITY_STARTDATE') }} as VALIDITY_STARTDATE,
    {{to_date_number_YYYYMMDD('VALIDITY_ENDDATE') }} as VALIDITY_ENDDATE
    
FROM
PRESTAGE_EMPLOYEES
```
