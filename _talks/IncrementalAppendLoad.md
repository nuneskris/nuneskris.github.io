---
title: "Incremental Append Only Load with Airbyte Demo"
collection: talks
permalink: /talks/IncrementalAppendLoad
date: 2024-03-01
venue: "Delta"
date: 2024-06-01
location: "Airbyte"
---

<img width="612" alt="image" src="https://github.com/user-attachments/assets/3b691ec4-9060-4302-b42e-60ce548bf610">

An append-only incremental upload is a data loading pattern where new records are continually added (appended) to a dataset without modifying or deleting existing records. This approach is often used when historical data must be preserved for audit, analysis, or other purposes.

Data is preserved and every new batch of data is added to the existing dataset without altering or removing past data. This is a simple method and is straightforward to implement since it avoids the complexity of handling updates and deletions. It is ideal for systems where data grows over time and past records need to be preserved. Very often we would need an audit trail for historical data analytics and this method maintains a complete history of all data, which is useful for auditing and tracking changes over time.
Common Use Cases for Append-Only Incremental Upload

# Set up.
Detailed [configuration and setup of Airbyte](https://nuneskris.github.io/teaching/Postgres-Airbyte-S3) and [incremental apppend + duplicate](https://nuneskris.github.io/talks/IncrementalLoadWithDemo) is already covered and I will be continuing from there.

The most import configuraiton is we need to identify the cursor similar to incremental apppend + duplicate, and setting the increment as append only.

<img width="612" alt="image" src="https://github.com/user-attachments/assets/27c6bd56-51c4-4b7b-b761-465cce524e78">

# Data Setup
We will be using a more simplyfied dataset with only 10 rows on a fresh database both at the Postgress source ans Snowflake target.

<img width="612" alt="image" src="https://github.com/user-attachments/assets/2b253edd-bfd8-4102-b903-14a7bd41979f">

# 1. Full Load

## Running the Airbyte Syc

<img width="612" alt="image" src="https://github.com/user-attachments/assets/2a823214-dfb4-441a-9f1c-d5c1b0079c46">

## Validating snowflake
We can see that there are 10 rows of the inital load which has been upoloaded into Snowflake.

<img width="612" alt="image" src="https://github.com/user-attachments/assets/80e31dd6-a9c6-4198-ab59-169c3a84c0fd">

<img width="612" alt="image" src="https://github.com/user-attachments/assets/ca5adf20-f8e4-4c24-8d1e-c048ad1d9349">

# 2. Inserted new rows.

<img width="612" alt="image" src="https://github.com/user-attachments/assets/c4e284b5-bd61-4ba3-8149-fd6ea7ec641f">

## Running Sync for the new rows

<img width="612" alt="image" src="https://github.com/user-attachments/assets/68f3d240-222a-4bdc-a7de-aa54654b889b">

## Validating the syc of the 2 new rows.
We can see a new syn job in Snowflake with job id 38.

<img width="612" alt="image" src="https://github.com/user-attachments/assets/f766f280-93ae-44a3-a0b3-3384c49fce49">

When we query for the row inserted with the job 38, we find the 2 rows which were loaded as expected.

<img width="612" alt="image" src="https://github.com/user-attachments/assets/1667938e-f448-439d-9857-7f9add4406e3">

We can also see there 10 original and 2 new rows, totalling 12 rows.

<img width="612" alt="image" src="https://github.com/user-attachments/assets/0b02d872-22e4-4dfa-a353-fe3b91a8e0b5">

# 3. Update/Delete Rows

## Data setup for Update/delete
<img width="612" alt="image" src="https://github.com/user-attachments/assets/c91feffc-f6f1-4e0a-9842-33e10abe7553">

## Updating Postgress Source

<img width="612" alt="image" src="https://github.com/user-attachments/assets/561e478b-ace3-4b87-8141-d8db0b8d159b">

## Running the Sync job for the Update/delete rows

<img width="612" alt="image" src="https://github.com/user-attachments/assets/97d8abf4-9766-45d4-bcaf-905e275ef2ed">

15 ## Validating the syc of the 2 new rows.
We can see a new syn job in Snowflake with job id 39.

<img width="612" alt="image" src="https://github.com/user-attachments/assets/57cd3973-da42-472a-8104-7614f7668c6c">

We can see the 2 rows. One for the update and one with the delete.

<img width="612" alt="image" src="https://github.com/user-attachments/assets/ec6a5bcf-0b90-457b-892b-3e64660ed4b7">

When we look the at count of rows, updates create new rows unlike the append+deduplicate patter.

<img width="612" alt="image" src="https://github.com/user-attachments/assets/1fe8369d-c473-4056-b40c-f9d6afb2e775">

* One row updating a anount values with ones
* One row to update the note as DELETE to indicate that the row is deleted.

<img width="612" alt="image" src="https://github.com/user-attachments/assets/ffda5ed8-74ce-46d1-b7bb-07e68021cb31">

Append-only incremental uploads are a simple and effective but this needs maintenance and validation regularly. By following best practices for efficient indexing, partitioning, data validation, backup and recovery, monitoring, and data retention we can run a stable extract and load.
