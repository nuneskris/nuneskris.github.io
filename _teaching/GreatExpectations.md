---
title: "Manage Data Quality With Great Expectations"
collection: teaching
type: "Data Quality"
permalink: /teaching/GreatExpectations
venue: "Snowflake"
location: "DBT Cloud"
date: 2024-06-01
---
<img width="613" alt="image" src="https://github.com/user-attachments/assets/c5e038a1-ec86-4c82-a71f-3afc7ea512da">

Great Expectations can have a learning curve simply because we do not know what to expect. I will use this page to demo how to install and get things started with Great Expectation.

Objectives:
1. Install
2. Code Setup in a python book
3. Creating a context which is the hook required into the API
4. Connecting to the source:  Snowflake Landing Database from a previous demo
5. Profiling to create generate baseinformation on the tables
6. Generating Documentation
7. Profiling for multiple tables
8. Updating Genterated Expectations
9. Adding Documentation Details

# 1.Install
Create a Python virtual environment and run.
```console
pip install great_expectations
```

# 2. Code Setup
Open a notebook from the virtual environment.
```python
import great_expectations as gx
gx.__version__
```
'0.18.19'

# 3. Context
```python
# we first need to create a context which is like a project in my mind. We can also provide a location where this project will be created under a folder gx inside it.
full_path_to_project_directory = '/Users/xxxxxx/Study/python/DataEngineering/ETL/Observe/kfnproject'
# The below will create a project
context = gx.get_context(project_root_dir=full_path_to_project_directory)
```
The key file is the great_expectations.yml which will hold the details of the project.

![image](https://github.com/user-attachments/assets/3754cc64-683a-4087-954c-a3ad8430ecfa)

# 4. Connecting to the data: Snowflake Landing
```python
# Connecting to a Snowflake database using a connection string based on the below
# my_connection_string = "snowflake://<USER_NAME>:<PASSWORD>@<ACCOUNT_NAME_OR_LOCATOR>/<DATABASE_NAME>/<SCHEMA_NAME>?warehouse=<WAREHOUSE_NAME>&role=<ROLE_NAME>"
# A DataConnector defines how to access data from your datasource. For SQL datasources like Snowflake, you might use a ConfiguredAssetSqlDataConnector or a RuntimeDataConnector.
connection_string = "snowflake://nuneskris:Gracesnow1982@wzb29778/DB_PRESTAGE/ERP?warehouse=compute_wh&role=ACCOUNTADMIN&application=great_expectations_oss"
datasource_config = {
    "name": "kfn_datasource",
    "execution_engine": {
        "class_name": "SqlAlchemyExecutionEngine",
        "connection_string": connection_string,
    },
    "data_connectors": {
        "default_inferred_data_connector_name": {
            "class_name": "InferredAssetSqlDataConnector",
            "name": "default_inferred_data_connector_name",
            "include_schema_name": True,
        },
    },
}

context.add_datasource(**datasource_config)
```
<great_expectations.datasource.new_datasource.Datasource at 0x1451bbb90>

```python
# List all data assets for the data connector. This should list all the tables within the database along with the Snowflake Schema they below to.
data_connector_name = "default_inferred_data_connector_name"  # Replace with your data connector's name
assets = context.datasources["kfn_datasource"].data_connectors[data_connector_name].get_available_data_asset_names()
print(assets)
```
['erp.addresses', 'erp.business_partners', 'erp.employees', 'erp.products', 'erp.product_categories', 'erp.product_texts', 'erp.sales_order', 'erp.sales_orders', 'erp.sales_order_items', 'erp_kris.sales_order', 'erp_kris_erp_etl.prestage_businesspartners', 'erp_schema.load_salesorders', 'erp_schema_erp_etl.prestage_addresses', 'erp_schema_erp_etl.prestage_businesspartners', 'erp_schema_erp_etl.prestage_employees', 'airbyte_internal.ERP_KRIS_raw__stream_sales_order', 'airbyte_internal.ERP_raw__stream_sales_order', 'airbyte_internal._airbyte_destination_state']

# 5. Profiling

```python
from great_expectations.core.batch import BatchRequest

# In Great Expectations, a batch request is a way to specify and retrieve a particular slice of data from your datasource for validation or profiling.
# A batch represents a specific subset of data, and the batch request provides the criteria for selecting that subset.
# What we need for a Batch Request:
# Datasource Name: The name of the datasource that holds the data you want to validate.
# Data Connector Name: A data connector provides the interface to interact with data stored in a particular location (like a database or file system) and determines how data is divided into batches.
# Data Asset Name: The specific table, file, or dataset you want to retrieve from the datasource.

from great_expectations.profile.user_configurable_profiler import UserConfigurableProfiler

# After configuring your datasource and data connector, you can use UserConfigurableProfiler or RuleBasedProfiler to generate base expectations automatically.
# UserConfigurableProfiler Class
# The UserConfigurableProfiler is a tool in Great Expectations that helps you automatically create a suite of expectations for your dataset. It profiles the dataset and generates common expectations based on the data's characteristics.
# This is particularly useful for quickly setting up a baseline suite of expectations that you can then refine and expand upon.

validator = context.get_validator(
    batch_request=BatchRequest(
        datasource_name="kfn_datasource",
        data_connector_name="default_inferred_data_connector_name",
        data_asset_name="erp.sales_order_items",
    )
)

profiler = UserConfigurableProfiler(profile_dataset=validator)

# In Great Expectations, an Expectation Suite is a collection of expectations that define the criteria for validating the quality and integrity of your data. 
# An expectation suite is like a test suite in software testing, but it applies to data, specifying the rules that your data should adhere to.

# What is an  Expectation Suite comprise of:
# Expectations: Each expectation in the suite is a specific rule or assertion about your data, such as "this column should not have null values" or "values in this column should fall within a certain range."
# Name: The suite has a name that uniquely identifies it within the context of your data validation workflows.
# Scope: An expectation suite can be designed to validate a specific dataset, a table, or even a specific subset of data (e.g., data from a particular time period).

suite = profiler.build_suite()
context.save_expectation_suite(suite, "kfn_suite")

```
<img width="735" alt="image" src="https://github.com/user-attachments/assets/2ae5eb6e-71d8-4409-bd6a-d660b92a8593">

We can that many expectations are generated.

![image](https://github.com/user-attachments/assets/880767fa-e913-449c-987b-862d3521d5b3)

There is a json file which is created with the name of the suite and the file creates all the expectations.

![image](https://github.com/user-attachments/assets/1384649c-b41a-4ded-b377-25bf3c577bf5)

# 6. Building/Generating Docs
```python
# Lets build the docs of this profiler
context.build_data_docs()
context.open_data_docs()
```
We can see a HTML file also created. It is possible to find the location of the documentation in the great_expectations.yml file.

![image](https://github.com/user-attachments/assets/d9bf50e7-ebf9-4a78-9a19-e342ef7d270d)

The HTML created below shows the details of the suite.

![image](https://github.com/user-attachments/assets/fcb147d9-d709-4dcd-a44d-bdfff007845d)


# 7. Profiling for multiple tables
```python
# List of tables in the erp schema
tables = ['addresses', 'business_partners', 'employees', 'products', 'product_categories', 'product_texts', 'sales_order', 'sales_order_items',] 

from great_expectations.profile.user_configurable_profiler import UserConfigurableProfiler

for table in tables:
    # Create a unique Expectation Suite name for each table
    suite_name = f"erplanding_{table}_expectation_suite"
    
    # Create a BatchRequest for each table
    batch_request = BatchRequest(
        datasource_name="kfn_datasource",
        data_connector_name="default_inferred_data_connector_name",
        data_asset_name=f"erp.{table}",
    )

    # Get the validator for the current table
    validator = context.get_validator(batch_request=batch_request)
    
    # Initialize the profiler with the validator
    profiler = UserConfigurableProfiler(profile_dataset=validator)

    # Build the Expectation Suite
    suite = profiler.build_suite()

    # Save the suite with the unique name
    context.save_expectation_suite(suite, expectation_suite_name=suite_name)

# Optionally build and view data docs
context.build_data_docs()
context.open_data_docs()
```

![image](https://github.com/user-attachments/assets/7cab3baa-a391-432b-bbb0-ce2ad496a85d)

# 8. Updating Genterated Expectations
Below is the addresses table expectation. As we can see there are multiple expectations which do is not applicable or needs to be updated.

<img width="612" alt="image" src="https://github.com/user-attachments/assets/234af738-731c-4956-8d6e-35002b4c3b41">

## 8.1 Remove expectation
```python
from great_expectations.core import ExpectationConfiguration

# Load the existing Expectation Suite
suite = context.get_expectation_suite(expectation_suite_name="erplanding_addresses_expectation_suite")

# Define the expectation to remove
config_to_remove = ExpectationConfiguration(
    expectation_type="expect_column_mean_to_be_between",
    kwargs={"column": "addressid"},
)

# Remove the specific expectation
suite.remove_expectation(
    config_to_remove, match_type="domain", remove_multiple_matches=False
)

# Save the updated suite
context.add_or_update_expectation_suite(expectation_suite=suite)

# Rebuild the Data Docs to see the changes
context.build_data_docs()
```

## 8.2 Update expectation

We will update max value to range to 1001000000

```python
updated_config = ExpectationConfiguration(
    expectation_type="expect_column_max_to_be_between",
    kwargs={
        "column": "addressid",
        "min_value":1000000073,
        "max_value":1001000000,
        "strict_max":False,
        "strict_min": False
    },
)

# Add the new expectation to the suite
suite.add_expectation(updated_config)

# Save the updated suite
context.add_or_update_expectation_suite(expectation_suite=suite)

```
Below we can that the expect_column_mean_to_be_between removed, expect_column_max_to_be_between updated
![image](https://github.com/user-attachments/assets/5e553378-3354-418c-b026-256ef9c606f1)

# 9. Adding Documentation Details: 

## 9.1 Column Documentation

Documentation of metadata is very important. Adding metadata to 

```python
updated_config = ExpectationConfiguration(
    expectation_type="expect_column_max_to_be_between",
    kwargs={
        "column": "addressid",
        "min_value":1000000073,
        "max_value":10010000,
        "strict_max":False,
        "strict_min": False
    },
    meta={"notes": "Description=This is the primary key of the table. We can add all the metadata in the table."
    },
)

# Add the new expectation to the suite
suite.add_expectation(updated_config)

# Save the updated suite
context.add_or_update_expectation_suite(expectation_suite=suite)
```

![image](https://github.com/user-attachments/assets/4be145bd-19d2-4e09-b81f-83223eadd4ce)

## 9.2 Table Level Documentation
We would like to add multiple lines into the documentation. I have achieved this by using a breakpoint.
```python
updated_config = ExpectationConfiguration(
    expectation_type="expect_table_columns_to_match_ordered_list",
    kwargs= {
        "column_list": [
          "addressid",
          "city",
          "postalcode",
          "street",
          "building",
          "country",
          "region",
          "addresstype",
          "validity_startdate",
          "validity_enddate",
          "latitude",
          "longitude"
        ]
      },
    meta={"notes": "Table Name=ERP.Addresses <br> Description=Contains address details of business partners <br> PrimaryKey=addressid <br> ForeignKeys=None"}
)
```
![image](https://github.com/user-attachments/assets/12abcb51-1ea6-4510-b59b-0db52981b566)


# Add the new expectation to the suite
suite.add_expectation(updated_config)

# Save the updated suite
context.add_or_update_expectation_suite(expectation_suite=suite)
```
