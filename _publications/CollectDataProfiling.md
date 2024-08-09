---
title: "Collect: Data Profiling"
collection: publications
permalink: /publication/CollectDataProfiling
excerpt: 'Data profiling is essential for understanding the quality, structure, and consistency of data'
date: 2024-05-01
venue: 'Processing'
tags:
  - Collect
---

Data profiling is an essential first step in understanding the quality, structure, and consistency of data. Profiling helps to reduce the risk of errors and failures in the ETL process by identifying and addressing potential issues upfront. Understanding data volume and complexity helps in optimizing ETL processes for better performance and resource utilization. Many projects fail to pay attention to this phase and struggle with surprises that could have been identified and planned for early on.

> Implementation Tip: Remember, this is a quick exercise to scope the time and effort required. Use data profiling tools that automate profiling. I've seen engineers build SQL scripts to profile data, wasting time and money. Most modern tools offer comprehensive profiling capabilities like statistical analysis, pattern recognition, and anomaly detection.

> Implementation Tip: The findings need to be reviewed and signed off by business and data owners to validate the understanding of the data context. The architect needs to be accountable for the deliverables from this phase. There may be more uncovered as we start building the pipelines, necessitating updates to the profiling results and regular reviews by stakeholders. Use the deliverables to communicate and collaborate with business stakeholders; it should not be hidden within the data engineering team.

There are 3 main types of information we profile for are
1. structural
2. content
3. relationship information 

Below is a template I have used to much success in multiple occasions.

## Dataset Information: 
Provide information about the dataset as a whole. It will include multiple tables.
* Dataset Name:  ex: Customer Data
* Source System Name:  ex: CRM
* Source System Data SME Name: Tom.Hardy@kfnstudyorg.com
* Source System Integration Maintenance Engineer Name: Iris.Murdoch@kfnstudyorg.com
* Amount of support needed by Source Integration Maintenance Engineer 
* Number of Tables to be extracted.
* Links to existing documentation of the source table.
* Date of Analysis:  ex: 2024-07-17
* Analyst Name:  ex: Kris Nunes

## Table Information
* Table Name:  ex: Customers
* Description:  ex: Contains customer details
* Primary Key:  ex: customer_id
* Foreign Keys: ex:  None
* Number of Rows:  ex: 100,000
* Number of Columns: ex:  12

## Column Information

| Column Name	| Data Type	 | Format/Pattern |	Null Values Allowed	| Unique Values	| Default Value	| Constraints	| Comments |
| --------    |--------    |--------        |--------              |--------       |--------      |--------      |-------- |
| customer_id	| INT		     |                | No	                 | Yes	         | None	        | PRIMARY KEY	  |         |
| first_name	| STRING	   |               |No                     | 	No         | None           |NOT NULL	      |         |
| last_name	  | STRING		 |               |No                    | 	No           | None         | NOT NULL	     |        |
| email	      | STRING	    | Email format	|No	                  |Yes	          |None	          |UNIQUE	       |          |
| phone_number	| STRING	 | (###) ###-####	| Yes	                 | No	           | None		      |               | Some missing values|
| birthdate	| DATE	        | YYYY-MM-DD	  | Yes	                | No	          | None		      |               | Some dates are in the future| 
| join_date	| DATE	       |YYYY-MM-DD	    |No	                 |No	           |CURRENT_DATE	   |NOT NULL	     |          |
| address	| STRING		| | Yes| 	No	| None		|  |  | 
| city	| STRING	| 	 |Yes| 	No	| None		|  |  | 
| state	| STRING	| 	 |	Yes| 	No	| None	|  |  | 
| zip_code	| STRING	| #####	| Yes	| No	| None		|  |  | 
| country	    | STRING	| 	No| 	No	| 'USA'	| NOT NULL | DEFAULT 'USA'	| | 	 |

### Numeric Columns
Provide summary statistics (mean, median, mode, range, standard deviation).
* Distribution of values (min, max, mean, standard deviation)
* Presence of outliers

### Categorical Columns
Provide frequency distribution and mode.
* Frequency distribution of categories
  * ex: state: 50 unique values (US states)
* Most common and least common values
  * ex: country: 1 unique value ('USA')

### Date/Time Columns. 
Date format issues plague ETL projects. Ensure this is handled early.
* Range of dates
* Frequency of dates
* Format and Pattern Analysis

  * ex : birthdate: Range from 1900-01-01 to 2024-07-01
  * ex : join_date: Range from 2010-01-01 to 2024-07-17

### String Columns
List any columns where data types are inconsistent or need standardization if standardization is required. Also ensure special charecters, new line charecters are all removed as they will cause issues downstream.
* Common patterns (e.g., email format, phone number format)
* Regular expressions used to identify patterns
  
  * ex : email: Valid email pattern, 5% of emails have invalid format.
  * ex: phone_number: (###) ###-#### format, 10% missing or malformed.

### Relationship Analysis
* Ensure integrity between tables by checking foreign key constraints.

### Foreign Key Relationships: Ensure integrity between tables by checking foreign key constraints.
Note any discrepancies in foreign key relationships.
* Parent Table and Column
* Child Table and Column
* Referential Integrity Constraints (e.g., CASCADE, SET NULL)

### Join Conditions:
* Common join conditions used with other tables
* Cardinality of relationships (one-to-one, one-to-many, many-to-many)
* Anomalies and Observations

### Missing Values: List columns with missing values and the percentage of missing values.
* Columns with missing values
* Percentage of missing values per column
  
  * ex : phone_number: 10% missing
  * ex : birthdate: 2% missing
  * ex : address, city, state, zip_code: 15% missing collectively.

###  Duplicate Values: Identify any duplicate rows or duplicate values in unique columns.
* Columns with duplicate values
* Number of duplicate rows
  * No duplicate primary key values (customer_id).

### Data Quality Issues: 
List all detected data inconsistencies and note any obvious errors in the data (e.g., negative values for age).
* Inconsistent data formats
* Invalid values (e.g., negative ages, dates in the future)
* Any other anomalies observed
  
  * ex : 5% of email values have an invalid format.
  * ex: Some birthdate values are in the future, indicating incorrect data.

## Closing Notes
* It is important for the data architect to summarize the the issues and provide an estimate on the effort required by downstream processing systems to fix the issues.
* Include Overall Data Quality and provide recommendations for improvement which will trigger detailed requirements. Suggest steps to improve data quality.
* Add Any additional observations such contraints enforced by the data, risks identified and any support needed to resolve the risks.
* Finally there needs to be clarity on who would be fixing issues. Source applications vs Data engineering team.
