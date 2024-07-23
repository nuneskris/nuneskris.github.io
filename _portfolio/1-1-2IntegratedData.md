---
title: "Integrated Data"
excerpt: "Combining data from different sources and providing a unified view <br/><img src='/images/portfolio/Integratedata.png'>"
collection: portfolio
---

<img width="612" alt="image" src="/images/portfolio/Integratedata.png">

Integrated data refers to the process of combining data from multiple busines functiona to provide comprehensive view of the business rather an an isolated view of a single business function. This process is crucial for organizations that need to analyze data from multiple systems or departments to make informed decisions. Integrated data ensures consistency, accuracy, and completeness by merging data into a cohesive dataset.

# The Challenge
Data is stored and managed in desperate data stores and are not automatically integrated together. 

The Same Data Is Defined Inconsistently Across Multiple Systems. Different systems are designed and built in isolation, storing data in native data model schemas and formats. A key challenge to integrating data is to relate data across these silos. Very often, the same data is defined inconsistently across multiple systems.  

<img width="612" alt="image" src="/images/portfolio/UnifiedDataExample.png">

A Sales process would refer to a customer within its database table, and to integrate the sales process with marketing, the customer identifiers in the tables of both the Sales system and the Marketing system need to match.

## Inconsistent Identifiers
A very common challenge is that different departments (Sales, Marketing, Customer Support) might use different customer IDs or formats for the same customer. For example, Sales might use "C12345," while Marketing uses "JD-5678."
## Redundant Systems Across Multiple Business Lines
It is very common to have customers managed separately in redundant systems across business lines. This often happens during mergers and acquisitions (M&As). There are multiple MDM projects specifically to deal with these integrations.
## Varying Data Formats
The address in the Sales database might be complete with street, city, state, and zip code, whereas the Marketing database might only store city and state. This inconsistency makes it difficult to match records accurately.
## Redundant and Conflicting Data
The same customer's name might be stored differently across departments (e.g., "John Doe" vs. "Doe, John"), leading to redundancy and potential conflicts when merging data.
## Different Data Attributes
Departments might store different attributes or details for the same customer. Marketing might store segmentation data that's not available in Sales or Support records.
## Data Quality Issues
Inaccurate or outdated information might be present in one department but not in another, leading to inconsistencies. For example, the phone number might be updated in the Customer Support system but not in the Sales system.

# The 7 dimensions of Data Integration
Data integration involves several challenges and considerations, especially when dealing with the complexities of modern data environments. These challenges can be categorized into several dimensions often referred to as the "7 Vs" of Data (including Big): Volume, Velocity, Variety, Variability, Veracity, Visualization, and Value.

For instance, ensuring that customer identifiers match across Sales and Marketing systems (Volume, Variety, and Veracity), processing customer data in real-time for personalized marketing campaigns (Velocity), and visualizing integrated customer profiles for better decision-making (Visualization) all contribute to achieving significant business value (Value).

## Volume
The sheer amount of data generated and stored across different systems can be overwhelming. Integrating large volumes of data from multiple sources requires robust infrastructure and scalable solutions.
## Velocity
The speed at which data is generated and needs to be processed is critical. Real-time or near-real-time data integration can be challenging, especially when different systems operate at different paces.
## Variety
Data comes in various formats and structures, from structured databases to unstructured text and multimedia. Integrating such diverse data types into a cohesive system requires sophisticated data transformation and normalization techniques.
## Variability
Data can vary in meaning and context, even within the same system. This inconsistency can lead to challenges in interpreting and integrating data accurately across different sources.
## Veracity
Ensuring the accuracy and reliability of data is paramount. Data from different systems might have varying levels of quality, leading to potential issues with trustworthiness and consistency.
## Visualization
Effectively visualizing integrated data is essential for making informed decisions. Tools and techniques for data visualization help in presenting complex, integrated data in an accessible and understandable format.
## Value
The ultimate goal of data integration is to derive meaningful insights and value from the data. This involves not only combining data but also analyzing it to extract actionable intelligence that can drive business decisions.
