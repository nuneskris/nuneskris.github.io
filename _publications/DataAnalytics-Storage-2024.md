---
title: "Storage in Data Analytics Storage in 2024"
collection: publications
permalink: /publication/DataAnalytics-Storage-2024
excerpt: 'Storage is foundational. The choices are simplified by the maturing technolgies. Providing a technical overview'
date: 2024-05-01
venue: 'Storage'
slidesurl: 'https://nuneskris.github.io/publication/Domain-Oriented-Business-Capability-Map'
paperurl: 'http://academicpages.github.io/files/paper1.pdf'
---

There is basically three components of the storage layer in a data analytics architecture. Where are we going to store it? Cloud Storage; How is the data stored? FileFormat; How do we interact with the stored data? Table Format. It is very important to know your storage layer. 

The choices are simple and the decision as to why choose one over the other is complex. The good news is we can choose all three based on preferences of each domains and move cleansed data around. There is a cost to this but we will get in this in another topic. However, irrespective of the cloud vendors (or a mix) which are chosen, below are the best practices into how we build the storage layer.

## Know your data architecture
More and more data is moving towards decentralization. By this, we may have solved a data swamp, but we need to be careful in loozing control on how data is stored within domains. I have written an article on how we define domains.

## Maintain Raw Data in Its Original Format.
When extracting data from source applications, it's crucial to maintain the data and file formats in their raw state. This approach ensures that the data is easily extractable and stays true to its source. Source application systems (tools, people, processes) are often not designed to manipulate data once it leaves the system. Instead of forcing these systems to process data into a prescribed format, focus on extracting the data efficiently in a manner that aligns with the source application's natural capabilities and limitations.

For example, in two separate organizations, data extraction was performed via REST APIs. This method proved to be slow and taxing on the source application, leading to delays and recovery difficulties. The data teams had standardized JSON as the data format for extraction, which may be appropriate for higher layers of the architecture but not ideal for initial extraction. The application teams were more than willing to use their existing REST APIs, but by directly querying the database for data extraction, performance improved by a factor of 20.

## Compress Data to Maximize Data Retention and Reduce Storage Costs

## Leverage Storage Classses to optimize cost

# Key Considerations

There needs to be clarity in the below foudational questions through the development and maintenance of the data analytics system.

## How much of data.
How much data is there to store data analytics sytems irrespective of the arhictecture involved (Data Lake,CDW, DLH, DataMesh, Data Fabric etc)? To answer this questions is tough. Very often we get quesions like, do we need to include replicated data across layers, data copies, archived data etc. The way I like to measure this is, going to the source applications and measure volume there. Use multiple ways to measure such as (1) Size (GB, TB), (2) number of tables, (3) average number of rows and columns (4) data growth. This gives a good idea on how much of data we are dealing with within a application, domain and enterprise.

## How do we want to oraganize data.

## Know enough about your data and their doamins.
A countinous discovery and assessment about what we know about the data we are managing is important to make decision across the various adata analytics archecture layers. 


## Based on the lifecyle of the data, leverage storage types offered by the cloud vendors (Tiers, Classes etc)
