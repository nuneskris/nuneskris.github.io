---
title: "ETL Data Test and Validate"
collection: teaching
type: "Data Warehouse"
permalink: /teaching/DBTTestAndValidation
venue: "Snowflake"
location: "DBT"
date: 2024-06-01
---

# Setup
1. Installed DBT Core locally. The install configuration can be described by the command: dbt debug
   ![image](https://github.com/user-attachments/assets/45b39419-c1a6-4d97-a4d1-f788200c5434)
 
2. Installed DBT Utils
   <img width="594" alt="image" src="https://github.com/user-attachments/assets/27d0cd44-4366-4471-9f41-44cb4d15b52c">
   
   <img width="535" alt="image" src="https://github.com/user-attachments/assets/acb88c05-5c0c-40f2-92a0-f97f37b7960d">


# Basic Test

These are the basic must-do test to validate data. Here are the common built-in tests you can use:

## Unique Test: Ensures that each value in the column is unique.
```yaml
- name: column_name
  tests:
    - unique
```

## Not Null Test: Ensures that no values in the column are null.
```yaml
- name: column_name
  tests:
    - not_null
```

## Accepted Values Test: Ensures that each value in the column is one of an allowed set of values.
```yaml
- name: column_name
  tests:
      - accepted_values:
          values: ['value1', 'value2', 'value3']
```

## Relationships Test: Ensures referential integrity between two tables, similar to a foreign key constraint.
```yaml
- name: column_name
  tests:
    - relationships:
        to: ref('other_table')
        field: other_column
```

### Expression is True Test: Ensures that a SQL expression is true for all rows.
```yaml
- name: column_name
  tests:
    - expression_is_true:
        expression: "column_name > 0"
```

