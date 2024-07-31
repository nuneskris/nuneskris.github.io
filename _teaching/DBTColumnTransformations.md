---
title: "Staging with Column Transformations"
collection: teaching
type: "DataLake"
permalink: /teaching/DBTColumnTransformations
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
11. Column Splitting: Split a column into multiple columns based on a delimiter.
```
SPLIT_PART(EMAILADDRESS, '@', 2) AS EMAILDOMAIN
```

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

```
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
