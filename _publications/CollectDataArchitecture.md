---
title: "Data Collection Architecture"
collection: publications
permalink: /publication/CollectDataArchitecture
excerpt: 'Keep the Raw Layer "Raw"'
date: 2024-05-01
venue: 'Processing'
tags:
  - Collect
---

There are 3 types of data loads our collect architecture would need to handle and they are determined by the sources of the data.
* Full load: Data extracted from sources as a complete full extraction. I have seen this happening for many reasons
  * The source system does not have the budget or the skills to be able build out an extraction of the increments (delta). 
  * The data volume at the source is small and not worth the effort to build out increamental extractions on changes.
  * The source system does not manage data with timestamp columns that identifies if data create, update, or delete.
* Delta load - Incremental data changes
  * The data volume at the source is large.
  * The source system maintains a timestamp field that identifies if data has been added, updated, or deleted.
  * The source system creates and updates files on data changes.
  * Your raw data lake is composed of your landing and conformance containers. Each container uses a 100% mandatory folder structure specific to its purpose.
* Streaming Data - TBD

### Maintain Raw Data in Its Original Format. No Transformation or Changes to Raw Data
* The primary goal is to ingest data quickly and efficiently into the raw layer, maintaining its original format without transformations.
* Extract data from source applications while keeping the data and file formats in their raw state.
* This ensures the data is easily extractable and remains true to its source. Avoid forcing source application systems to process data into prescribed formats.
* Instead, focus on efficient extraction that aligns with the source application's capabilities. For example, in two separate organizations, data extraction via REST APIs proved slow and taxing. Switching to direct database queries improved performance by a factor of 20.
* A good rule of thumb is, we need to be able to easily build the data from the raw layer to form the data within the source system. If we are not able to do this, then we have complicated the raw layer.

### Maintain data archives at this layer, serving as a rollback point for any processing. 
* Establish clear policies for how long data should be retained based on regulatory requirements and business needs. This is often overlooked and there are many situations where were this is caught in audit and it gets expensive to fix this at a later point.
* Use automation to archive data once they are processed by down stream transformations. Most cloud storage provides this feature to move dara into cold storage.
* Implement tiered storage solutions to balance cost and access speed, using lower-cost storage for older, less frequently accessed data. The lifecyle management offered out-of-the box by cloud vendors are excellent place to start.
* Periodically test data recovery processes to ensure archives are usable and retrievable when needed. This needs to be a policy set on datateams on how the schedule and publication of results. It is not to much effort and needs to be done by the AMS teams.
* As part of maintaining the archived data, use checksums and other integrity checks to ensure archived data remains uncorrupted over time.
* Ensure archival processes comply with relevant regulations and maintain audit trails for accountability and traceability. These audit trails need to be seperately saved and published. There logs created by the Cloud vendor tools should suffice. This is typically done when compliance mandates this.

### Restrict end-user access to this layer
* This raw data is immutable. Since raw layer is only used for landing the data and for down stream applications to extract data from it, there should not be any write access by any consumers of the raw layer. Keep your raw data locked down, there should be read-only access by the consumers.
* The only write/delete access whould be producers of data and archival automation.
* Protect archived data with encryption and access controls to prevent unauthorized access and ensure compliance with data protection regulations.

### Documentation and Metadata
* An often forgotten aspect of data engneering is not documenting data in the raw layer. Maintain thorough documentation and metadata for archived data to facilitate easy retrieval and understanding of the context.
* Modern catalogs automate this aspect but we still forget this.
* Meta data should also include the extraction job details such as load type, design consideration etc.

### Use automatic policies to compress and archive data to reduce costs. 
* If data is not compressed at this layer, compress it before archival. Use automation for the same. Trigger the serverless functions offered by cloud vendors for this.

### Handle duplicates.
TBD

### Organizing the Raw Layer
* Within each domain, segregate data based on the source system and further partition it based on the time of ingestion to facilitate archiving and retrieval.
* Use multiple tags on ingested objects, such as the time of batch ingestion, to calculate cutoff times.
* Organize this layer by using one folder/bucket per source system. Give each ingestion process write access to only its associated folder. One interesting aspect I have noticed is, alerts are at a bucket level which makes it easier to build automation on the bucket.
* Use proper date partitions on the data at the raw layer and use additional tagging on the file/folder/bucket.
