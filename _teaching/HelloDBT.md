---
title: "Hello DBT"
collection: teaching
type: "Datawarehouse"
permalink: /teaching/HelloDBT
venue: "S3"
date: 2024-06-01
location: "Snowflake"
---
Installation notes
1. The below is what I always use to setup DBT Cloud on Snowflake
* The main configuration is the default schema: ERP_SCHEMA
https://docs.getdbt.com/guides/snowflake?step=4
2. Configure DBT on Github if needed.


# Column Level Transformations

1. Renaming Columns: Change column names for consistency or clarity. ``` LOGINNAME as USERNAME,```
2. Column Data Type Conversion: Changing data type
3. Column Merging: Combine multiple columns into one.
4. Handling Missing Values: Fill, drop, or impute missing values : 

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

I have additionally used macros for the column transformation: A dbt macro is a reusable piece of code written in Jinja, a templating language. Macros help automate repetitive tasks, making your dbt project more efficient and maintainable. You can think of a macro as a function that you can call with specific arguments to perform a task or generate code dynamically.

```
-- macros/to_date.sql
{% macro to_date_number_YYYYMMDD(column) %}
  TO_DATE(CAST({{ column }} AS STRING), 'YYYYMMDD')
{% endmacro %}

-- macros/concat_columns.sql
{% macro concat_columns(columns, separator=' ') %}
    CONCAT_WS('{{ separator }}', {{ columns | join(', ') }})
{% endmacro %}
```






Column Splitting:
Split a column into multiple columns based on a delimiter.
Example: Split full_name into first_name and last_name.


Example: Merge first_name and last_name into full_name.

Derived Columns:
Create new columns based on existing ones.
Example: Calculate total_price from quantity and unit_price.

Column Filtering:
Remove unwanted columns from the dataset.
Example: Drop columns not needed for analysis.


Example: Fill missing values in age column with the median age.
String Transformations:
Perform operations like trimming, padding, and case conversion.
Example: Convert product_name to lowercase.
Aggregations:
Perform aggregate functions like sum, average, min, max on columns.
Example: Calculate the average salary for each department.
Conditional Transformations:
Apply transformations based on conditions.
Example: Assign a category based on a value range in the score column.
Column Normalization/Standardization:
Normalize or standardize column values for consistency.
Example: Scale price column values to a range of 0-1.
Pivoting and Unpivoting:
Reshape data by pivoting or unpivoting columns.
Example: Pivot month columns to rows.
Calculating Running Totals or Differences:
Compute cumulative totals or differences between rows.
Example: Calculate the running total of sales.
Regular Expression Transformations:
Use regex to transform column values.
Example: Extract domain from an email address.
Date and Time Transformations:
Extract parts of dates, convert time zones, or format dates.
Example: Extract year from a date column.
