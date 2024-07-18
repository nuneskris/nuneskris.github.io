---
title: "Cloud Storage: Best practices"
collection: talks
type: "Talk"
permalink: /talks/CloudStorage-Best-Practices
date: 2024-02-01
---

* Buckets names: I am split on wheter to have have smart names which clear inform about the intent of the bucket and its files and the security concerns that may arise by doing so. If there is a need to hide the intent of buckets from possible attackers, we would need manage and enforce catalogs. However, I have seen the worst of both worlds in which the naming is gives enough and these buckets not being cataloged. I would recommend a naming coventions or rules to catalog bucket names and have audits to ensure compliance.

* Single purpose buckets: Do not use size as a reason to create buckets. The number of objects in a bucket does not impact performance. The purpose of these storage c

Use separate buckets for S3 data which needs to be replicated.
* Production buckets should be hosted under a different production accounts, separate from non-production workloads. Isolatation is key to security and avoiding leakage. In data as a product paradigm, provision sandbox storage buckets with seperate non production security and SLAs.

Deleted bucket names are not available to reuse immediately; hence, if users want to use a particular bucket name, they should not delete the bucket.

All bucket names across all AWS Regions should comply with DNS naming conventions.

Build an automatic ingestion mechanism to catalog and create the multiple layers of data storage including Raw, Transformed, and Curated.

Consider building automatic data classification rules based on schema and data.

Consider additional folders within the data lakes, such as reports, downstream applications, or user folders.

Enable versioning, if needed, for protection from accidental deletes.
