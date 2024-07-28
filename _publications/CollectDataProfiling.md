---
title: "Data Profiling"
collection: publications
permalink: /publication/CollectDataProfiling
excerpt: 'Data profiling is essential for understanding the quality, structure, and consistency of data'
date: 2024-05-01
venue: 'Processing'
tags:
  - Integration
  - Collect
---

Data profiling is essential begining step in understanding the quality, structure, and consistency of data. Profiling helps to reduce the risk of errors and failures in the ETL process by identifying and addressing potential issues upfront. Understanding data volume and complexity helps in optimizing ETL processes for better performance and resource utilization.
Avoid Rework: Profiling helps avoid rework by identifying issues early, preventing the need for repeated ETL runs due to unexpected data issues.I have seen many projects not paying attention to this phase and struggling with many surprises which could have been identified and planned at an early. Remember this is a quick exercise and scope the time and effort which we would want to spend for this. Use data profiling tools that automate profiling the data. I have seen engineers building SQL scripts to profile data and burning time and money. Most mordern tools offer most profiling capabilities like statistical analysis, pattern recognition, and anomaly detection. 

The deliverable findings need to be reviewed and signed off by business and data owners to validate the understanding the context of the data, and the architect needs to be accountable on the findings of the deliverables from this phase. There may be more we uncover as we start builing the pipelines, and we need to go back and update the profiling results and have the results regularly reviewed by the stakeholders.

Review and Refine: 

The dataset contains various data quality issues, including missing values, invalid email formats, and future birthdate entries.

## Business Context Analysis

Business Relevance:
Describe how the dataset fits within the broader business context.
Usage Scenarios:
Identify potential use cases for the data.
Stakeholders:
List the stakeholders who are interested in or affected by the data.

## Dataset Information: Provide information about the dataset as a whole. It will include multiple tables.
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
* Include Overall Data Quality and provide recommendations for improvement which will trigger detailed requirements. Suggest steps to improve data quality (e.g., data cleaning, validation rules).
* Add Any additional observations or noteworthy points regarding the dataset.
* Create a Data Dictionary: Document the structure, types, and constraints of the data.
Profile Reports: Maintain detailed reports of profiling results, including identified issues and their potential impact.
