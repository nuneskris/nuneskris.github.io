---
title: "EMR Serverless - Finally"
collection: teaching
type: "Lakehouse"
permalink: /teaching/AWS-EMR Serverless-Hello
date: 2024-06-01
venue: "EMR"
date: 2024-06-01
location: "AWS"
---
This is a continuation of the setup. 

Below are the list of Iceberg which are listed by AWS and I would like to test and demonstrate them.

1## Delete, update, and merge. Iceberg supports standard SQL commands for data warehousing for use with data lake tables.

Fast scan planning and advanced filtering. Iceberg stores metadata such as partition and column-level statistics that can be used by engines to speed up planning and running queries.

Full schema evolution. Iceberg supports adding, dropping, updating, or renaming columns without side-effects.

Partition evolution. You can update the partition layout of a table as data volume or query patterns change. Iceberg supports changing the columns that a table is partitioned on, or adding columns to, or removing columns from, composite partitions.

Hidden partitioning. This feature prevents reading unnecessary partitions automatically. This eliminates the need for users to understand the table's partitioning details or to add extra filters to their queries.

Version rollback. Users can quickly correct problems by reverting to a pre-transaction state.

Time travel. Users can query a specific previous version of a table.

Serializable isolation. Table changes are atomic, so readers never see partial or uncommitted changes.

Concurrent writers. Iceberg uses optimistic concurrency to allow multiple transactions to succeed. In case of conflicts, one of the writers has to retry the transaction.

Open file formats. Iceberg supports multiple open source file formats, including Apache Parquet, Apache Avro, and Apache ORC.


