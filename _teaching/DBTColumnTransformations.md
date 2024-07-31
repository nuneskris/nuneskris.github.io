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
