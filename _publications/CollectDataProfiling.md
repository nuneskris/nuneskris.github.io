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


# Structural Analysis Template

### Dataset Information

* Dataset Name: Customer Data
* Source System: CRM
* Date of Analysis: 2024-07-17
* Analyst Name: John Doe

### Table Information

* Table Name: Customers
* Description: Contains customer details
* Primary Key: customer_id
* Foreign Keys: None
* Number of Rows: 10,000
* Number of Columns: 12

Column Information

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


  
