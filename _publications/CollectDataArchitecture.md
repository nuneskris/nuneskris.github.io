---
title: "Data Collection Architecture"
collection: publications
permalink: /publication/CollectDataArchitecture
excerpt: ''
date: 2024-05-01
venue: 'Processing'
slidesurl: ''
paperurl: 'http://academicpages.github.io/files/paper1.pdf'
---

# Raw Layer Architecture: Keep the Raw Layer "Raw"
## Maintain Raw Data in Its Original Format. No Transformation or Changes to Raw Data
The primary goal is to ingest data quickly and efficiently into the raw layer, maintaining its original format without transformations.
Extract data from source applications while keeping the data and file formats in their raw state. 
This ensures the data is easily extractable and remains true to its source. Avoid forcing source application systems to process data into prescribed formats. 
Instead, focus on efficient extraction that aligns with the source application's capabilities. For example, in two separate organizations, data extraction via REST APIs proved slow and taxing. Switching to direct database queries improved performance by a factor of 20.
A good rule of thumb is, we need to be able to easily build the data from the raw layer to form the data within the source system. 
If we are not able to do this, then we have complicated the raw layer.
## Maintain data archives at this layer, serving as a rollback point for any processing. 
Implement data archival strategies for long-term storage of historical data using cost-effective solutions for infrequently accessed data needed for compliance or historical analysis. 
Regularly test backup and recovery procedures.
## Restrict end-user access to this layer
## Use automatic policies to compress and archive data to reduce costs. 
## Handle duplicates and different versions of data without allowing overrides.
## Organizing the Raw Layer
Within each domain, segregate data based on the source system and further partition it based on the time of ingestion to facilitate archiving and retrieval. Use multiple tags on ingested objects, such as the time of batch ingestion, to calculate cutoff times.
