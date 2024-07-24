---
title: "What is usable data"
excerpt: "we need it to have an acceptable quality, readily available and easily accessible."
collection: portfolio
date: 2024-01-01
venue: 'Vision'
---

For data to be usable for analytics purposes, we need it to have an acceptable quality, readily available and easily accessible. 

# Disoverable
But for data consumers who are not savvy with to interact with the data, we would need to go even further to make it usable. I appreaciate Data Mesh for articulating just that. They start with data needing to be Discoverable. Essentially, users would need to centralized regitry (catalog) where they can browse what data is available with enough information about the dataset for them to know if it would be applicable to their needs. 

We can imagine multiple interesting information which can be provided to the user for them to assess if the data meets their needs such as sample dataset, context of the business it would be applicable. For example, a customer 360-degree view data can provides all the aspects of the customer which are avaible within the dataset such as contact information, purchase history, and support interactions. We categorize this data which we provide to the users as ***Descriptive (or business) Metadata*** and includes information on title, purpose, business intent, creation date, creator, key entity definitions, detailed descriptions of important relationships between entities and rules that govern those relationships from a business perspective. Typically we would include classification data covering security, privacy (public, private, restricted or regulatory classifications such as PII).

Data stewards or business SMEs who can speak the language consumers can relate to are typically the ones who provides this data (metadata) which support the disoverability of the data within a catalog. There are a few standards such as Dublin Core and MARC. Most mordern data catalog tools support this level of metdadata. 

# Understandable
Once the user is able to know that the data which they have disovered to support their business needs, we would need to provide further information to the users for them to understand the data so that they can interact with it. We typicall would provide ***Structural Metadata*** to understand the schema of the dataset, how data is formatted, with information on tables, views, entity types, and relationships. 

It can additional provide information to the users to understand how data can be organized and providing guidance on how the data can be combined with other data with links to types of queries that can be executed on the datasets, code snippets, demonstractions on the data, comments from other users and major usecases where the datasets have been used 

# 

Before I go any further there is a great online resource which publishes the [FAIR](https://www.go-fair.org/fair-principles/) principles for data. 
> provides guidelines to improve the Findability, Accessibility, Interoperability, and Reuse of digital assets. The principles emphasise machine-actionability (i.e., the capacity of computational systems to find, access, interoperate, and reuse data with none or minimal human intervention) because humans increasingly rely on computational support to deal with data as a result of the increase in volume, complexity, and creation speed of data.
 


Data Mesh refers to Findability as Addressable. But it also 
