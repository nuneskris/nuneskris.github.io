---
title: "Curate: Data Cleansing"
collection: publications
permalink: /publication/CurateDataCleansing
excerpt: 'TBD'
date: 2024-05-01
venue: 'Processing'
tags:
  - Curate
---

# 1. Column Standardization
Format Normalization is key to accelerate development velocity and quality code. We would need to convert data into a common format (e.g., dates to a standard format, converting all text to lowercase).
This also includes ***data type conversions** to appropriate types (e.g., string to date, float to integer). Below are common examples

#### Renaming Columns
Change column names for consistency or clarity.
```sql
 LOGINNAME as USERNAME
```

#### Column Data Type Conversion
Changing data type is one of the most common column transformations.
```sql
{{to_date_number_YYYYMMDD('VALIDITY_STARTDATE') }} as VALIDITY_STARTDATE
```

# 2. Column Transformation
Very often we are required to parsing column text (Strings) to splitting or extracting parts of data (e.g., extracting domain from email, splitting full name into first and last names).
Also we often would need to combine or merge text from multiple columns to into a single. This will include simple String split based on a char or a regex expression.

#### Column Merging
Combine multiple columns into one. I once had to combine columns for a text processing usecase.
```sql
CONCAT_WS(' ', NAME_FIRST, NAME_MIDDLE, NAME_LAST, NAME_INITIALS) as full_name
```

#### Regular Expression Transformations
Use expressions on string values
```sql
REGEXP_SUBSTR(
	REGEXP_REPLACE
		(  WEBADDRESS, 'https?://|www\.|/$', ''), '^[^/]+') AS EXTRACTED_DOMAIN
```

#### Column Splitting
Split a column into multiple columns based on a delimiter.
``` sql
SPLIT_PART(EMAILADDRESS, '@', 2) AS EMAILDOMAIN
```

# 3. Consistent Casing and Spacing
Another common text processing on columns is converting text data to a consistent case (e.g., all lowercase or all uppercase).  Trimming leading, trailing, or excessive in-between spaces.
#### String Transformations
Perform operations like trimming, padding, and case conversion.

```sql
   UPPER(CITY) AS CITY
```

#### Trimming Column Text: Medium Descito
```
TRIM({{ column }})
```

# 4. New Column Imputation
Derive and create a new columns based on existing data to either improve on the existing column.

#### Derivation
Date and Time Transformations by extracting parts of dates, convert time zones, or format dates.
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

#### Conditional Transformations
Apply transformations based on conditions.
```sql
	CASE 
	WHEN ADDRESSTYPE = 1 THEN 'BILL'
        WHEN ADDRESSTYPE = 2 THEN 'SHIP'
        ELSE CAST(ADDRESSTYPE AS STRING)
	END AS ADDRESSTYPE_TRANSFORMED
```

#### Handling Missing Values
Fill, drop, or impute missing values
```sql
coalesce(NAME_FIRST, '') as NAME_FIRST
```
#### PENDING Augmentation: Adding additional data from external sources to enrich the dataset (e.g., appending geographic information based on IP addresses).

#### Column Value Imputation
Imputing a column value of Short Description from Medium Desc column only when the Medium Desc column is not null.
```sql
CASE
	WHEN MEDIUM_DESCR IS NOT NULL THEN {{ trim_text('MEDIUM_DESCR') }} 
	 ELSE  {{ trim_text('SHORT_DESCR') }}
END AS DESCRIPTION
```

# Data Validation
Type Checking: Ensuring data types are correct (e.g., integers, dates).
Range Checking: Ensuring values fall within a specified range (e.g., age between 0 and 120).
Format Validation: Ensuring data follows a specific format (e.g., phone numbers, email addresses).

# Handling Missing Values:
Imputation: Filling in missing values using various strategies (mean, median, mode, previous/next value).
Deletion: Removing records with missing values, if they are not critical.
Flagging: Marking records with missing values for further review or special handling.

# Data Deduplication:
Identifying Duplicates: Finding and identifying duplicate records using various techniques (exact match, fuzzy matching).
Merging Records: Combining duplicate records into a single record.
Removing Duplicates: Deleting redundant duplicate records.

# Data Masking:
Anonymization: Masking or anonymizing sensitive data (e.g., replacing names with pseudonyms, masking credit card numbers).
Redaction: Removing or hiding sensitive data (e.g., removing social security numbers).

# Data Integrity Checks:
Referential Integrity: Ensuring relationships between tables are maintained (e.g., foreign key constraints).
Unique Constraints: Ensuring that specific columns contain unique values (e.g., email addresses, primary keys).

# Geocoding:
Standardizing Addresses: Converting addresses to a standard format.
Geocoding: Converting addresses to geographic coordinates (latitude and longitude).

Aggregating: Summarizing or aggregating data (e.g., total sales per month).
Joining: Merging data from multiple sources or tables (e.g., joining customer data with order data).

9. Column Filtering: Remove unwanted columns from the dataset.

Data Correction:
* Error Correction: Correcting known errors in data (e.g., fixing typos, correcting known incorrect values).
* Outlier Handling: Identifying and handling outliers (e.g., reviewing and correcting, flagging, or removing).
