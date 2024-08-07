---
title: "Column Transformations for Staging"
collection: talks
type: "Lakehouse"
permalink: /talks/ColumnTransformations
venue: "DBT, Snowflake"
location: "AWS"
date: 2024-07-01
---

<img width="300" alt="image" src="https://github.com/user-attachments/assets/062203d2-9899-4635-af22-51546adc694f">

# Objectives

#### Column Level Transformations
1. Renaming Columns: Change column names for consistency or clarity.
```sql
 LOGINNAME as USERNAME
```
2. Column Data Type Conversion: Changing data type
```sql
{{to_date_number_YYYYMMDD('VALIDITY_STARTDATE') }} as VALIDITY_STARTDATE
```
3. Column Merging: Combine multiple columns into one.
```sql
CONCAT_WS(' ', NAME_FIRST, NAME_MIDDLE, NAME_LAST, NAME_INITIALS) as full_name
```
4. Handling Missing Values: Fill, drop, or impute missing values
```sql
coalesce(NAME_FIRST, '') as NAME_FIRST
```
5. Regular Expression Transformations: Use expressions on string values
```sql
REGEXP_SUBSTR(
	REGEXP_REPLACE
		(  WEBADDRESS, 'https?://|www\.|/$', ''), '^[^/]+') AS EXTRACTED_DOMAIN
```
6. Conditional Transformations: Apply transformations based on conditions.
```sql
	CASE 
	WHEN ADDRESSTYPE = 1 THEN 'BILL'
        WHEN ADDRESSTYPE = 2 THEN 'SHIP'
        ELSE CAST(ADDRESSTYPE AS STRING)
	END AS ADDRESSTYPE_TRANSFORMED
```
7. String Transformations: Perform operations like trimming, padding, and case conversion.
```sql
   UPPER(CITY) AS CITY
```
8. Column Filtering: Remove unwanted columns from the dataset.
9. Date and Time Transformations: Extract parts of dates, convert time zones, or format dates.
```sql
CAST(SUBSTRING(CAST(FISCALYEARPERIOD AS STRING), 1, 4) AS INTEGER) AS FISCALYEAR,
CAST(SUBSTRING(CAST(FISCALYEARPERIOD AS STRING), 5, 3) AS INTEGER) AS FISCALMONTH,
CASE
    WHEN CAST(SUBSTRING(CAST(FISCALYEARPERIOD AS STRING), 5, 3) AS INTEGER) IN (1, 2, 3) THEN 1
    WHEN CAST(SUBSTRING(CAST(FISCALYEARPERIOD AS STRING), 5, 3) AS INTEGER) IN (4, 5, 6) THEN 2
    WHEN CAST(SUBSTRING(CAST(FISCALYEARPERIOD AS STRING), 5, 3) AS INTEGER) IN (7, 8, 9) THEN 3
    ELSE 4
END AS FISCALQUARTER
```
10. Column Splitting: Split a column into multiple columns based on a delimiter.
```sql
SPLIT_PART(EMAILADDRESS, '@', 2) AS EMAILDOMAIN
```
11. Column Value Imputation: Imputing a column value of Short Description from Medium Desc column only when the Medium Desc column is not null.
```sql
CASE
	WHEN MEDIUM_DESCR IS NOT NULL THEN {{ trim_text('MEDIUM_DESCR') }} 
	 ELSE  {{ trim_text('SHORT_DESCR') }}
END AS DESCRIPTION
```
12. Trimming Column Text: Medium Descito
```
TRIM({{ column }})
```


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

#### Performed a Regular Expression Transformation to remove the domain form the 

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

## ProductText Table - Text Processing
* Column Value Imputation: Imputing a column value of Short Description from Medium Desc column only when the Medium Desc column is not null.
* Trimming Column Text: Medium Description is trimmed for leadning.

![image](https://github.com/user-attachments/assets/f9b65273-f549-44d5-a7c0-17fabbc4ddda)

Used a macro to handle trimming.
```
 TRIM({{ column }})
```
```sql
-- models/src/erp/salesorderitems/prestage_businesspartners.sql
-- Column Value Imputation: Imputing a column value of Short Description from Medium Desc column only when the Medium Desc column is not null.
-- Trimming Column Text: Medium Descito
{{
  config(
    schema='erp_etl'
  )
}}
WITH PRESTAGE_PRODUCT_TEXTS AS ( SELECT
    *,
        CASE
            WHEN MEDIUM_DESCR IS NOT NULL THEN {{ trim_text('MEDIUM_DESCR') }} 
            ELSE  {{ trim_text('SHORT_DESCR') }}
        END AS DESCRIPTION
    ,
FROM
DB_PRESTAGE.ERP.PRODUCT_TEXTS
)
SELECT
    PRODUCTID,
	LANGUAGE,
	DESCRIPTION
FROM
PRESTAGE_PRODUCT_TEXTS
```
### Output
![image](https://github.com/user-attachments/assets/07e68e5c-c139-4f21-92c2-d43d10bc12c7)

# Other Tables
### Products
```sql
-- models/src/erp/products/prestage_products.sql
{{
  config(
    schema='erp_etl'
  )
}}
WITH PRESTAGE_PRODUCTS AS ( SELECT
    *
FROM
DB_PRESTAGE.ERP.PRODUCTS
)
SELECT
    PRODUCTID,
	TYPECODE,
	PRODCATEGORYID,
	CREATEDBY,
	{{to_date_number_YYYYMMDD('CREATEDAT') }} AS CREATEDAT,
	CHANGEDBY,
    {{to_date_number_YYYYMMDD('CHANGEDAT') }} AS CHANGEDAT,
	SUPPLIER_PARTNERID,
	TAXTARIFFCODE,
	QUANTITYUNIT,
	WEIGHTMEASURE,
	WEIGHTUNIT,
	CURRENCY,
	PRICE,
	WIDTH,
	DEPTH,
	HEIGHT,
	DIMENSIONUNIT,
	PRODUCTPICURL
    
FROM
PRESTAGE_PRODUCTS
```
