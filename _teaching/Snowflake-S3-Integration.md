/*--
In this Worksheet we will walk through templated SQL for the end to end process required
to load data from Amazon S3,
Helpful Snowflake Documentation:
        1. Bulk Loading from Amazon S3 - https://docs.snowflake.com/en/user-guide/data-load-s3
-*/
-------------------------------------------------------------------------------------------
    -- Step 1: To start, let's set the Role and Warehouse context
        -- USE ROLE: https://docs.snowflake.com/en/sql-reference/sql/use-role
        -- USE WAREHOUSE: https://docs.snowflake.com/en/sql-reference/sql/use-warehouse
-------------------------------------------------------------------------------------------

--> To run a single query, place your cursor in the query editor and select the Run button (⌘-Return).
--> To run the entire worksheet, select 'Run All' from the dropdown next to the Run button (⌘-Shift-Return).

---> set Role Context
USE ROLE accountadmin;

---> set Warehouse Context
USE WAREHOUSE compute_wh;
-------------------------------------------------------------------------------------------
    -- Step 2: Create Database
        -- CREATE DATABASE: https://docs.snowflake.com/en/sql-reference/sql/create-database
-------------------------------------------------------------------------------------------
---> create the Database
CREATE  OR REPLACE  DATABASE  db_prestage;
-------------------------------------------------------------------------------------------
    -- Step 3: Create Schema
        -- CREATE SCHEMA: https://docs.snowflake.com/en/sql-reference/sql/create-schema
-------------------------------------------------------------------------------------------
---> create the Schema
CREATE SCHEMA IF NOT EXISTS db_prestage.ERP
   COMMENT = 'Loading data from S3' ;
-------------------------------------------------------------------------------------------
    -- Step 4: Create Table
        -- CREATE TABLE: https://docs.snowflake.com/en/sql-reference/sql/create-table
-------------------------------------------------------------------------------------------
---> create the Table
CREATE  TABLE  IF NOT EXISTS  db_prestage.ERP.business_partners
    (
   PARTNERID INTEGER,
    PARTNERROLE INTEGER,
    EMAILADDRESS varchar,
    PHONENUMBER INTEGER,
    FAXNUMBER varchar,
    WEBADDRESS varchar,
    ADDRESSID INTEGER,
    COMPANYNAME varchar,
    LEGALFORM varchar,
    CREATEDBY INTEGER,
    CREATEDAT INTEGER,
    CHANGEDBY INTEGER,
    CHANGEDAT INTEGER,
    CURRENCY varchar
    --> supported types: https://docs.snowflake.com/en/sql-reference/intro-summary-data-types.html
    )
    COMMENT = 'Creating a table';

---> query the empty Table
SELECT * FROM db_prestage.ERP.business_partners;
