---
title: "MDM Patterns. All are relevant and can coexist"
collection: publications
permalink: /publication/Integrate-MDMPatterns
excerpt: 'Leverage Business Capability Maps for Data Domains'
date: 2024-05-01
venue: 'Master Data'
tags:
  - Integration
  - Master Data
---

Understanding the four main MDM patterns goes a long way toward building a viable architecture for managing master data. I am seeing Master Data architecture being simplified with business transaction processing applications maturing in their MDM capabilities.

However, when we assess the current landscape of an enterprise, most of the MDM implementations would fall within these four main patterns.

<img width="612" alt="image" src="/images/publications/MDMPatterns.png">

# Consolidation Pattern
Most of the legacy MDM implementations I have seen fall into this pattern. The consolidation pattern centralizes master data from multiple sources into a single hub, creating a golden record through data cleansing, matching, and merging. The main driver was to improve data quality to create a single version of truth for the master data, which was then used to further support analytics initiatives.

However, I have seen these implementations struggle to keep up with data integration as source systems evolve and reconciling the source systems with the cleansed record, especially when there are frequent changes or inconsistencies in the source systems.

# Transaction Pattern
Over the years, business applications started improving their MDM capabilities and MDM transactions (Create/Read/Update/Delete), and these features would be integrated into the business process the application supports (e.g., SAP MDG). These systems would then push down curated records to downstream consumers. This was ideal as long as there was strong centralization of the source of truth of the data entity within the single application supporting the MDM transactions.

They typically did not try to create golden records by matching and merging with other sources of the master data entity. This high dependency tends to break down as soon as the organization starts growing, with parallel business applications supporting various business lines emerging due to mergers, acquisitions, and new product line rollouts.

# Coexistence Pattern
To resolve the issues with the above two patterns (reconciling the master record with the source systems and recognizing the existence of applications managing large volumes of MDM transactions), the coexistence pattern synchronizes with the source system using the curated record and, most importantly, synchronizes responsibilities. For example, if there is a key transactional MDM along with a few peripheral sources of master data, then including a central MDM for matching and merging, having golden records reconciled with the sources. There are multiple combinations we can come up with in this pattern.

## Regardless of the above three patterns, below are the main challenges:
* Changes in Data Formats and Structures: Continuous updates to integration processes are required to accommodate changes in data formats, structures, or new data sources. These updates are costly and need thorough planning and realistic cost estimation.
* Reconciling Source Data: Reconciling source data with the cleansed golden record is challenging due to implementation constraints in source applications, especially when there are frequent changes or inconsistencies.
* Matching and Merging: This process is resource-intensive and requires significant subject matter expertise. High levels of sponsorship and stewardship are crucial.

# Registry Pattern
Recognizing the challenges as a reality, we need to accept that multiple source and MDM systems can exist within an organization, each at varying levels of maturity in mastering records and reconciliation. The registry pattern involves creating a central registry that indexes master data records from various source systems without physically moving the data.

## Key Components of a Registry MDM

<img width="612" alt="image" src="/images/publications/RegistryMDM.png">

The core component of the registry MDM is the ***Registry Database***. I have used a document-oriented database to effectively design this component. Design the database to manage the following:

* Unique Identifiers: Manage unique identifiers (index) for master data records from various source systems, source system references, and additional metadata around the identifier. Assign smart unique identifiers to each master data record to facilitate mapping and linking.
* Data Definitions and Formats: Store data definitions and formats for each source system.
* Data Lineage: Maintain data lineage information to track the origin and transformations of data.
* Governance Policies: Store governance policies and data quality rules.

The key capability of the Registry MDM is ***Data Mapping and Linking***. This enables the registry MDM to map the unique identifier within the registry database to the source managing the master data and link to that record physically stored at the source to create a unified view. Often, sources of master data manage different aspects of the master data. Variants in master data occur when different source systems have different versions or representations of the same entity, such as customer preferences versus customer profiles. Below are the features we would need to support to map and link data:

* Define rules to link records that represent the same entity but have different attributes or values.
* Group linked records together to form a composite view of the entity, preserving variations from source systems.
* Define a canonical data model that represents the ideal structure and format of master data.
* Map attributes from source systems to the canonical model, handling variations in data formats and structures.
* Implement rules to standardize data formats and values across source systems.
* Normalize data to a common format before mapping it to the registry.

Optionally, we can have ***Data Matching*** capabilities. I recommend buying an AI-driven tool to identify and match records across source systems. Define matching criteria to identify duplicate or related records across source systems. Implement fuzzy matching algorithms to handle variations in data, such as misspellings or different formats. Use a scoring system to determine the likelihood of matches and set thresholds for automatic and manual review. I have used Tamr, which was great for this.

Additionally, we would need ***APIs and Connectors***. These interfaces connect to source systems for real-time or batch data access. Implement APIs and connectors to allow real-time access to master data from the registry. Support batch processing for initial data loading and periodic updates. Ensure that changes in source systems are synchronized with the registry in real-time or at scheduled intervals. I have used fast cache databases which mapped to the sources, were refreshed, and would publish master data from the sources.

# Best Practices for Overcoming Challenges
* Agile Data Integration: Implement agile integration methods to quickly adapt to changes in source systems. Use modern data integration platforms that support real-time or near-real-time data synchronization.
* Incremental Updates: Use incremental data loading techniques to update only the changed data, reducing the integration load and latency.
* Continuous Data Quality Monitoring: Implement ongoing data quality monitoring and automated correction mechanisms to maintain the integrity of the golden record.
* Flexible Data Models: Design flexible and extensible data models in the central repository to accommodate changes in source data structures.
* Data Lineage and Auditing: Maintain comprehensive data lineage and auditing capabilities to track the origin and transformations of data, facilitating easier reconciliation and compliance.
* Stakeholder Collaboration: Foster close collaboration between IT and business stakeholders to ensure alignment on data definitions, quality standards, and governance policies.

Selecting the right MDM implementation pattern is crucial for the success of an MDM initiative. Each pattern has its own advantages and challenges, and the choice should be guided by the organization's specific requirements, existing infrastructure, and long-term data management goals. By carefully evaluating these factors, organizations can implement an effective MDM solution that ensures data consistency, quality, and governance across the enterprise.
