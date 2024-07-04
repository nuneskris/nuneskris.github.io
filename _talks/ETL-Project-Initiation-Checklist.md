---
title: "ETL Project Initiation Checklist"
collection: talks
type: "Tutorial"
permalink: /talks/ETL-Project-Initiation-Checklist
date: 2019-03-01
---

## Acquisition
* Are the data sources external to the cloud.  If this is the case, we need an ingestion design (compressed/encrypted data from external sources) and add complexity to the overall solution.
* Irrespective of the extraction pattern (Delta, Full Load), it is recommended to use a data file landing stage on cloud storage which is key to roll-back, archival (raw data) and audit.
* Establish standard tools and SLAs with extraction/source teams which highly impact timelines and quality of the first time delivery.
* Data encrypted in-flight/at-rest. Also is the security architecture built around acquiring the data and a regular audit process to ensure compliance
* Establishing the frequency at which data is to be extracted from the sources to the system. This will be based on entity type, reporting needs and source system constraints. This needs be handled case-by-case basis and one size does not fit all. This needs to be designed along with the source system.
* Is it possible to extract delta-loads from the source system.
* It is recommended to extract only desired data from the source systems when we start so that we don’t get overwhelmed. Do we have a list of tables which need to be extracted.
* What is the format of data which will be extracted (type, compression, size etc.)
* How much reliability do we need to build into the extraction.

## Transformation/Processing
* What are the stages (Logical Partitions) do we have within the Data lake. We would need clear partitions on the various stages so that there is separation on computing needs, security and risk profile.
* Is there any data which is personal or sensitive that should be anonymized or concealed to protect data privacy and ensure regulatory compliance.
* Do we have clear data cleansing requirements, or do we need to run any data-profiling tools to scan the data? (missing or inaccurate values, poorly structured fields, Accuracy, Completeness, Consistency, Timeliness, Validity, Uniqueness).
* Do we have a canonical data model to represent the main subject areas? What is the data model impedance between the data source model and the target canonical model.
* How many types of consumers do we have for the single source of truth (canonical data)
* What are the decision arguments behind the technology stack for processing data?
* The dimensional model.
* Is there an estimate on the volume of data which needs to be processed. The performance consideration which needs to be addressed accordingly.
* Is there a design to handle slowly changing dimensions. The extraction of slowly changing dimensions will impact on the SCD type.
* Do we need to maintain a data catalog or schemas. Do we have a mapping between the final reports to tables?
* Do we have governance requirements which need to be included?
* Reliability should be built into the pipelines.

## Serving
* How many reports and do we have clear requirements?
* Is there a logical grouping of the reports from a development perspective.
* Do we have a mapping between reports and data requirements.
* The serving data model should be mapped using the rule of thumb. One query to one table.
* Separate self-service/ ad-hoc reporting from high frequency operational reporting. 
