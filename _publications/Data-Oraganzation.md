---
title: "Measure Data Architecture"
collection: publications
permalink: /publication/Data-Organization
excerpt: 'Consistency on what we measure and how we measure data domains.'
tags:
  - Governance
---

<img width="923" alt="image" src="https://github.com/user-attachments/assets/4ad5c6b2-69c1-4339-81bd-5f44db2a1f52">


During my Master’s thesis, I focused on measuring architecture programs within enterprises, which ingrained in me the importance of measurement in all aspects of work. While most metrics may not perfectly capture reality or predict outcomes with complete accuracy, they are certainly better than having no metrics at all.

[In a page on Data Domains](https://nuneskris.github.io/publication/Domain-Oriented-Business-Capability-Map), I recommended building data domains aligned with business capability maps to support federated data management and governance. However, it’s crucial to ensure consistency in what we measure and how we measure these data domains.

As part of a large transformation program, I was tasked with developing a framework to assess our current position and create a roadmap for where we need to go. I quickly realized that this was a politically sensitive endeavor. Rather than relying solely on financial metrics, which are typically used at a portfolio level, we needed a method to provide a more holistic perspective on data management.

To achieve this, I employed a four-point scale to evaluate various dimensions of the architecture, as well as its lifecycle stages, from the current state (as-is) to full implementation.

Below is an example of a data domain and some of the dimensions which we typically manage in any enterprise.

<img width="964" alt="image" src="/images/publications/datadomain.png">

# Current State Assessment

|-------- |-------- |-------- |
| Data Collection | 2 | The company collects basic data on inventory levels, inbound and outbound shipments, and warehouse capacity utilization. However, data from warehouse management systems (WMS) is not fully integrated with enterprise systems. |
| Data Quality | 3 | There are inconsistencies in inventory data due to manual data entry errors and lack of real-time updates, leading to discrepancies in stock levels. |
| Analytics Infrastructure | 3 | The company relies on legacy systems with limited integration capabilities. Most data is stored in siloed databases, and analytics is performed using basic SQL queries and Excel spreadsheets. |
| Descriptive Analytics | 1 | Basic reports are generated on warehouse performance metrics, such as order fulfillment rates, but these reports are often outdated and lack actionable insights. |
| Predictive Analytics | 1 | Predictive analytics capabilities are nonexistent. The organization does not forecast demand or optimize warehouse layouts based on data-driven insights. |
| Data Collection | 3 | Decisions in warehouse operations are predominantly made based on past experiences and intuition rather than data-driven insights. There is a lack of data literacy among warehouse management and staff. |





The company collects supplier systems and customer data from a loyalty program. However, data from online channels and social media is not systematically collected or integrated.
## Data Quality: 
There are frequent issues with data accuracy and completeness, especially with customer data, leading to inconsistencies in reports.
## Analytics Infrastructure: 
The company uses on-premise databases and spreadsheets for data analysis. There is no centralized data warehouse, and the existing infrastructure struggles with processing large datasets.
## Descriptive Analytics: 
Basic descriptive analytics are performed using Excel, focusing mainly on historical sales reporting. There is limited use of dashboards, and reports are manually generated.
## Predictive Analytics: 
Predictive analytics capabilities are minimal. There are no machine learning models in place to forecast demand or personalize marketing efforts.
## Data Culture: 
The organization relies heavily on intuition and experience for decision-making. Data-driven culture is not fully embraced, and there is limited data literacy among non-technical staff.
