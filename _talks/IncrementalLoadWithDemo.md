---
title: "Incremental Load with Airbyte Demo"
collection: talks
permalink: /talks/IncrementalLoadWithDemo
date: 2024-03-01
venue: "Delta"
date: 2024-06-01
location: "Airbyte"
---
<img width="666" alt="image" src="https://github.com/user-attachments/assets/dbdfc4e6-ed1c-41cc-b1ec-f29d7abe7a96">

The objective of this demo is to be able to query a source table in Postgres, perform an extract from the source table and load in a desination analytics environment. However this would to most importantly perform only delta changes.
* Full Load
* Insert New Records
* Update Change Records
* Update With Delete Change

# Setup

Please refer a Sanbox demo to set up [Postgres, Airbyte and Snowflake](https://nuneskris.github.io/teaching/Postgres-Airbyte-S3).

Updating the date audit columns to timestamp for the purpose of this demo. Used excel formula for preping the data
```
=TEXT(DATE(LEFT(A1, 4), MID(A1, 5, 2), RIGHT(A1, 2)) + TIME(12, 0, 0), "yyyy-mm-dd HH:MM:SS") & ".000"
```

<img width="666" alt="image" src="https://github.com/user-attachments/assets/741fff01-aac9-47ae-861f-b03fc9ce80b5">



<img width="666" alt="image" src="https://github.com/user-attachments/assets/b305513c-ae76-47cc-98ee-03b5bd77e4c9">


