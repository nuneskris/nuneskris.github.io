---
title: "Data Processing with DBT and Snowflake"
collection: teaching
type: "Data Warehouse"
permalink: /teaching/DBTColumnTransformations
venue: "Snowflake"
location: "DBT Cloud"
date: 2024-06-01
---
<img width="354" alt="image" src="/images/teachings/snowflake/DBTCleanse.png"> 

I will be Using DBT to run some tranformations on Snowflake. I will be using the ERP data which I have used in multiple demos. 
Data was loaded into Snowflake stages from this [demo](https://nuneskris.github.io/teaching/Snowflake-S3-Integration).

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

## First Model
The First Model which sources and creates the first staging layer which focuses on Column Transformation
* Renaming Columns: LOGINNAME as USERNAME
* Handling Missing Values: Names. coalesce: Replace null values with a specified value (empty string in this case) to prevent nulls from affecting concatenation.
* Column Merging: merging names into a single column
* Column Data Type Conversion: casting interger date types into a date
#### Cleaning up Addresses

* Column Filtering: STREET BUILDING
* Conditional Transformations: ADDRESSTYPE
* String Transformations: CITY
<img src='/images/teachings/snowflake/FirstDBTMacro.png'>
I have additionally used macros for the column transformation: A dbt macro is a reusable piece of code written in Jinja, a templating language. Macros help automate repetitive tasks, making your dbt project more efficient and maintainable. You can think of a macro as a function that you can call with specific arguments to perform a task or generate code dynamically.

<img src='/images/teachings/snowflake/firstdbtmodel.png'>
## First Run
A view is created in Snowflake when we run the above model.

<img src='/images/teachings/snowflake/FirstDBTRun.png'>
