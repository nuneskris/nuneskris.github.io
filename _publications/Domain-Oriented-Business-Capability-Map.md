---
title: "Federated Data Management through Domain-Oriented Decentralized Data Ownership"
collection: publications
permalink: /publication/Domain-Oriented-Business-Capability-Map
excerpt: 'Leverage Business Capability Maps for Data Domains'
---

A federated approach to data management, characterized by domain-oriented decentralized data ownership and architecture, represents a relatively new paradigm that addresses many limitations of traditional centralized data management. This approach promotes scalability, flexibility, and responsiveness, and was a core aspect of the data strategy I worked on in 2017.

This concept is championed by the Data Mesh Architecture, which prescribes a Domain-Oriented Decentralized Data Ownership and Architecture. In this model, data is owned by the domain that best understands it, rather than a central IT team. Each domain (e.g., sales, marketing, finance) manages its own data as a product, taking responsibility for its quality, governance, and lifecycle.

There are multiple ways to define these domains and organize data around them. However, the driving force behind decentralizing to data domains is ownership, management, and governance. Leveraging TOGAF guidance can help in structuring and governing this architecture effectively. 
> An Enterprise Architecture imposed without appropriate political backing is bound to fail. In order to succeed, the Enterprise Architecture must reflect the needs of the organization. Enterprise Architects, if they are not involved in the development of business strategy, must at least have a fundamental understanding of it and of the prevailing business issues facing the organization. It may even be necessary for them to be involved in the system deployment process and to ultimately own the investment and product selection decisions arising from the implementation of the Technology Architecture. (TOGAF on Architecture Governance)

# Key considerations

There are many considerations when we developn a data architecture. Specifically for Federated Data Managemen, below are key
***Domain Ownership*** Data ownership is decentralized to the domain level. Domains that generate and utilize the data are best suited to manage it.
***Data as a Product*** Each domain treats its data as a product. Domains are responsible for data quality, governance, and lifecycle management.
***Scalability and Flexibility*** Decentralized data management scales more effectively. Flexible to adapt to the unique needs and expertise of each domain.
***Governance and Architecture***  Ensure a robust framework for data management and governance across domains.

Capability is an abstract concept by design and misunderstood by me in the beginning. I have seen this tool used to great value in many places, and a money pit in a few. However it is by far the most accepted artifact by the business and a great tool to gain a clear understanding of the functional areas and domains. They serve as a foundation for decision-making related to business transformation, process improvement, organizational restructuring, and IT initiatives.

Mapping Data Domains to organization departments directly can be a political nightmare and messy. However, BCMs have already solved a similar problem by providing a cross-functional perspective, meaning they span across different departments or functional areas of the organization. Organizations have come to understand and buy into these BCMs and recognize how departments interact and depend on each other through capabilites.

I find business capability typically corresponds to specific data domains or areas of data ownership. For example, a "Customer Management" ***capability aligns and maps*** with data domains related to customer data, such as customer master profiles and and customer preferences. Within each business capability we would then add a new dimension to define ***data ownership responsibilities***. Data ownership includes accountability for data quality, accuracy, security, and compliance within that domain.

I have some main takeways on successfully working with Business Capability Maps.

>Implementation Tip: Embrace the principle Good is good enough. Work our way up from an industry model and don’t get caught in a paralysis for perfection. It should be left at a strategic planning level and not worry too much of decomposing capabilities down too much.

>Implementation Tip: Being an abstract concept it does not need to change too often. Hence this is ideal to align relatively volatile organization structures, Programs, IT Functions etc to each other.

>Implementation Tip: Use the capability map in conjunction with other strategic planning tools such as Target Operating Models, Long term planning and other strategic analysis tools. By relating the capability map to other tools adds relevance and insights to the overall strategy planning.

>Implementation Tip: Focus on specific and tangible outcomes. The capability map and can be multiple things but never all at once. It can be both a tool for M&A as one of the respondents suggested or it can be used to optimize your IT portfolio [invest buy vs invest build vs outsource]. Having it do both, will add complexity and leave a bad taste to the mouth.

>Implementation Tip: Value is in the insights and not how pretty it looks. So rather than focusing on tooling and canvas focus on how we can articulate data and measures. Simple heat mapping can go a long enough way.

Understanding Usage Patterns of organzations behind these business capabilities provides a framwork to organize domains and their data. Below is based on my experiences. 

Sales and Marketing
* Data Volume: Often substantial due to customer data, sales transactions, marketing campaigns, social media interactions, and CRM systems.
* Types of Data: Customer demographics, purchase history, campaign metrics, web analytics.
* Trend: Increasing use of big data analytics, customer segmentation, and personalized marketing drives data volume.

Finance
* Data Volume: Moderate to high, depending on the scale of transactions and regulatory requirements.
* Types of Data: Financial transactions, accounting records, investment data, regulatory compliance data.
* Trend: Data volume grows with detailed financial analysis, real-time transaction processing, and compliance reporting.

Operations and Supply Chain
* Data Volume: High due to logistics, inventory management, manufacturing processes, and IoT data.
* Types of Data: Supply chain transactions, inventory levels, production data, shipment tracking, sensor data.
* Trend: Use of IoT and real-time tracking systems increases the volume and complexity of data.

Human Resources (HR)
* Data Volume: Lower compared to sales and operations but still significant.
* Types of Data: Employee records, payroll data, performance metrics, recruitment data.
* Trend: HR analytics and employee performance tracking add to data volume.

Engineering
* Data Volume: Variable, often high in the sectors I worked in (Aeroa and manufacturing).
* Types of Data: Experiment results, simulation data, product testing data, innovation metrics.
* Trend: High volume in sectors reliant on continuous innovation and development.
