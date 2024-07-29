---
title: "Data Collection Architecture"
collection: publications
permalink: /publication/DataEngineeringBestPractices
excerpt: ''
date: 2024-05-01
venue: 'Processing'
tags:
  - Collect
---
1. Design time improvements
    1. End to end data engineering teams
    2. Extract necessary data only: The entire pipeline gets stressed when we extract more data than needed. 
    3. Secure data both at rest and transfer
    4. Ensure data quality at source
2. Build time improvements
    1. Breaking larger jobs into smaller ones
    2. Compress data whenever possible (Transfer and Archival)
    3. Manage error and reject record handling: Review if we can handle rejected records separately rather than failing the entire job.
    4. Review portioning of data for parallel or bulk processing
        1. A Salesforce Load component which took 7 hours was designed with a realtime load vs bulk load which improved the load time by 10 fold.
3. Runtime improvements
    1. Automate manual interventions across the pipeline
        1. There was a 4 hour job which had 8 stages under a single AutoSys sequence. These jobs would fail individually this forced the support engineers to keep monitoring each job fix an issue and trigger the subsequent jobs. Simple solution of having separate Austosys routine for each data stage job which is triggered from the end of the subsequent job, with delay alarms. 
    2. Appropriate logging, monitoring and alerting
    3. Strive to move maintenance tasks to L1. Ex: Files not arriving, Restart jobs etc.
