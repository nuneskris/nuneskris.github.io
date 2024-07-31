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

# Installation notes
1. The below is what I always use to setup DBT Cloud on Snowflake
* The main configuration is the default schema: ERP_SCHEMA
https://docs.getdbt.com/guides/snowflake?step=4
2. Configure DBT on Github if needed.

# Hello DBT

We will staging data from the src by performing column transformtions on the Employee table This is the inputdata

<img width="300" alt="image" src="https://github.com/user-attachments/assets/062203d2-9899-4635-af22-51546adc694f">

## Using the default package.
The first run will have only one model which will be a src extraction.

<img width="300" alt="image" src="https://github.com/user-attachments/assets/062203d2-9899-4635-af22-51546adc694f">
