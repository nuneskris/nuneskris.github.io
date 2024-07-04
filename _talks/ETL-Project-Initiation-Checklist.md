---
title: "Data Engineering Project Initiation Checklist"
collection: talks
type: "Tutorial"
permalink: /talks/ETL-Project-Initiation-Checklist
date: 2019-03-01
---

Some upfront work is required to ensure the success of data engineering projects. I have used this checklist to provide a framework for collaborating with multiple stakeholders to define clear requirements and designs.

## Acquisition

* Are the data sources external to the cloud? If so, we need an ingestion design (compressed/encrypted data from external sources) which adds complexity to the overall solution.
* Regardless of the extraction pattern (Delta, Full Load), it is recommended to use a data file landing stage on cloud storage for roll-back, archival (raw data), and audit purposes.
* Establish standard tools and SLAs with extraction/source teams that significantly impact timelines and the quality of initial deliveries.
* Is data encrypted in-flight/at-rest? Is the security architecture built around acquiring the data, with a regular audit process to ensure compliance?
* Establish the frequency of data extraction from sources to the system based on entity type, reporting needs, and source system constraints. This should be handled on a case-by-case basis as one size does not fit all.
* Can delta-loads be extracted from the source system?
* When starting, it's recommended to extract only the necessary data from source systems to avoid overwhelm. Do we have a list of tables that need extraction?
* What format (type, compression, size, etc.) should the extracted data be in?
* How much reliability needs to be built into the extraction process?
  
## Transformation/Processing

* What stages (Logical Partitions) do we have within the Data lake? Clear partitions are needed for computing needs, security, and risk profile separation.
* Is there personal or sensitive data that needs anonymization or concealment to protect privacy and ensure regulatory compliance?
* Are there clear data cleansing requirements? Do we need to use data-profiling tools to scan for missing or inaccurate values, poorly structured fields, and ensure Accuracy, Completeness, Consistency, Timeliness, Validity, Uniqueness?
* Do we have a canonical data model to represent main subject areas? What is the data model impedance between the data source model and the target canonical model?
* How many types of consumers are there for the single source of truth (canonical data)?
* What are the decision criteria for selecting the technology stack for processing data?
* Is there a dimensional model in place?
* Do we have an estimate of the volume of data to be processed? What performance considerations need to be addressed accordingly?
* Is there a design to handle slowly changing dimensions? This impacts the SCD type.
* Do we need to maintain a data catalog or schemas? Is there a mapping between final reports and tables?
* Are there governance requirements that need to be included?
* Reliability should be built into the pipelines.

## Serving

* How many reports do we have, and are the requirements clear?
* Is there a logical grouping of reports from a development perspective?
* Do we have a mapping between reports and data requirements?
* The serving data model should follow the rule of thumb: one query to one table.
* Separate self-service/ad-hoc reporting from high-frequency operational reporting.
