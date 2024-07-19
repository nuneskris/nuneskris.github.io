---
title: "Data Platform - Enteprise Semantic Layer Requirements"
collection: publications
permalink: /publication/Data-Platform-Semantic-Layer-Requirements
excerpt: 'Deliver Data as an Organized, Unified and Consistent Product'
date: 2024-5-01
venue: 'Data Platform'
slidesurl: 'http://academicpages.github.io/files/slides2.pdf'
paperurl: 'http://academicpages.github.io/files/paper2.pdf'
---
This layer is by far the most complex which leads to disagreement among data community and difficult to get buy-in as this effects the entire enterprise. Decentralization is awesome, but they need to meet somewhere so that data can be consistently integrated accross the domains and accessed in an unified model by data consumers enterprise wide.

I will have this as an open document which I believe I would keep refining. Note that, many of my other pages would be zooming into the topics listed below.

The semantic layer connects the producers and consumers of data. It is an enterprise wide layer that integrates and conforms the cleansed data logically from multiple data domains. A key objective of this layer is to abstracts consumers from source systems and elevates the data into an Enterprise context .How can we deliver data in an organized, unified, and consistent way that it can be easily understandable and accessible to end users. 

# First Principles
Data Mesh has laid out the first principles of Data as a Product which I will guidance from.

* Understandability.
* Valuable on its own
* Trusworthy
* Addressability
* Discoverability

# Requirements
Here are the key requirements and considerations for the semantic layer in a data analytics system:

## Requirement: Business-Friendly Unified Data Models
The semantic layer should provide business-friendly abstractions of raw data, transforming complexity, technical data structures into understandable and meaningful business concepts. The Semantic Layer needs to be modeled such that the domains are integrated into a enterprise context and they should consolidate commonalities and accommodate exclusivities of source system/process variations, providing a comprehensive and inclusive framework for the data. This can only be achieved with clear articulation of relationships accross entities. Articulation should be focused on the ***Understandability** of the data by the consumer.

### Strategies:
* Mordern source systems are investing heavily on the data models keepijng downstream analytics in mind. Leverage existing source data model.
* Data Domains own the data models defininions within the domain and there needs to be consensus building on relationships with entities across data domains. Data Governance is key to measure how well the models conform accross domains

## Requirement: Quality, Clean and Managed Data Source of Truth
Data needs contracts between producers of data and consumers of data. This contract needs to be governed. It’s crucial that the data is accurate captures business operations. These contracts would need to include timeliness, refresh rate completeness, lineage, statistical and range information, performance etc. Ensure data is ***trustworthy***.

### Strategies:
* Metadata Repository: Maintain a metadata repository that stores information about data definitions, business rules, transformations, and lineage. This helps in managing and accessing metadata efficiently. Keep metadata synchronized with the underlying data sources and transformations to ensure that it accurately reflects the current state of the data.
* Data Lineage to track and visualize data lineage to understand the flow of data from source to destination. This helps in ensuring data quality and transparency.
* Implement data governance policies and practices to manage data quality, data stewardship, and compliance. This includes defining data ownership and stewardship roles.

## Requirement: Consistency and Standardization 
 
***Addressability*** means that data can be uniquely identified and accessed. This is typically achieved through unique identifiers and URIs (Uniform Resource Identifiers) which are globally understood and refered to when integrating datya (logically) from various data domains.
****Discoverability*** ensures that data consumers can easily find the data they need. 
### Strategies:
* Every data product should have a unique address that helps data consumers access it programmatically. The address typically follows centrally decided naming standards within the organization. Globally Unique Identifiers (GUIDs): Use GUIDs or UUIDs for unique identification of data records. There needs to be a balance as to what entities require global identifiers. If an entitiy is used within the context of the domain, this would be an overkill. Metadata Management Systems: Tools like Apache Atlas or OpenMetadata can help assign and manage URIs and GUIDs. Data Catalogs: Systems like Alation or Collibra to provide a central repository where each data asset is uniquely identifiable.
* User standard storage formats, fileformats, table formats, API interfaces. Centrally managed self service data platforms
* Maintain standardized metadata that includes data definitions, business rules, and transformations. This helps in ensuring that users and applications interpret the data in the same way.
* Uniform Resource Identifiers (URIs): Assign URIs to datasets and data products to ensure they can be uniquely identified, located and and accessed.This involves maintaining a comprehensive catalog and providing search and indexing capabilities. Register all entities which are published in this layer as part of the data as a product in the layer. A standard location is recommended.
* Implement robust search capabilities within the data catalog to allow users to find data assets quickly. Use tags and metadata to enhance searchability. Include descriptions, keywords, and business context.

## Requirement: User-Friendly Self-Service Interfaces
Provide user-friendly interfaces that allow business users without needing deep technical knowledge to either ingest the data via simple data extraction tools or interact with the data directly for creating reports, dashboards, and ad-hoc queries. Thus this layer should be able to seemlessly integrate with consuming tools. [Natively Accessable]

## Requirement: Performance, Scalability and Flexibility
Design the semantic layer to scale with the growth of data and user demands (Volume). This includes handling large volumes of data and concurrent user queries efficiently. Ensure that the semantic layer can adapt to changes in data sources, business requirements, and technology. This includes supporting changes in data models and business logic (Schema Evolution). Implement mechanisms to optimize query performance.

### Strategies:
* Since data needs to be udpated, we wouuld model the data in the 3rd normal form.
* Ensure the data incorporates type 2 slowly changing dimensions.
* Store data in file formats such as Parquet or ORC as they are columnar, support partioning and not size intensive.
* Use table format such as Iceberg, Hudi or Delta Tables as they support ACID transactions, evolve schema etc. (Modern Data Lakehouse)

## Requirement: Interoperability

## Requirement: Data Integration and Federation
Unified View of Data: Integrate data from various sources and present it in a unified manner. This often involves creating views or virtual tables that combine data from multiple sources.
Data Federation: Support querying and analysis across different data sources and formats without requiring data movement. This is crucial for leveraging data distributed across different storage systems.

## Requirement: Security and Access Control
Data Security: Implement robust security measures to protect sensitive data. This includes encryption, access controls,  and auditing.
Fine-Grained Access Control: Provide fine-grained access controls to ensure that users only have access to the data they are authorized to see. This includes row-level and column-level security.

The semantic layer in a data lakehouse plays a crucial role in enabling users to interact with data in a meaningful and efficient way. It provides a business-friendly view of data, ensures consistency and accuracy, optimizes query performance, and supports self-service analytics. By addressing these requirements, organizations can create a robust and effective semantic layer that enhances data accessibility, usability, and value.
