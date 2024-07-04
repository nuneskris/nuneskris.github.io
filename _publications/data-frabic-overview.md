---
title: "Data Fabric in 2 mins"
collection: publications
permalink: /publication/data-frabic-overview
excerpt: '
What? Enterprise wide consisted data management design.
Why? Reduce the time to deliver data integration and interoperability
How? Through metadata.'
date: 2024-02-17
---

Simply explained
*      What? Enterprise wide consisted data management design.
*      Why? Reduce the time to deliver data integration and interoperability
*      How? Through metadata.

Gartner has been pushing this for a while and I have condensed 2 key points in their own words.
> *A data fabric maps data residing in disparate applications (within the underlying data stores, regardless of the original deployment designs and locations) and makes them ready for business exploration.*
> *A data fabric utilizes continuous analytics over existing, discoverable and inferenced metadata assets to support the design, deployment and utilization of integrated and reusable data across all environments, including hybrid and multi-cloud platforms.*

# My thoughts.

## Integrated Logical Data

At its core, I believe data fabric aims to ***reduce the time needed to integrate data***. Data warehouses integrate data from various sources to provide a unified view for analysis, with approaches such as Inmon, Kimball, Data Lake, Lakehouse, and Logical Data Warehouse attempting to achieve this goal. Data fabric leverages metadata on logical data to accomplish this integration more efficiently. I led the development of the architecture and platform for a very large program that adopted data fabric as its data strategy. Below are the data management capabilities required to achieve this.
*        1. *Describe* what the data is. To effectively manage and integrate data, we need a Unified Data Model. This model consolidates commonalities and accommodates exclusivities, providing a comprehensive and inclusive framework for our data. Modern data catalogs excel at managing data models through automation, streamlining this process significantly.

The key is knowing what level of detail is sufficient. Overmodeling can cause initiatives to lose momentum and become overly complex. I've observed many projects falter at this stage due to excessive modeling. It's essential to strike a balance, ensuring the data model is detailed enough to be useful but not so intricate that it hinders progress.

2. *Organize* Unified Data. is when many fragmented data sources are merged into one logical data set based on the unified data model.

3. Share:  
    * Uniform Interface: Data across the enterprise needs to be accessed via a uniform interface.
4. Integrate:  
    * Semantic Model with self described Data Integration: 
        * Hypermedia with linked relationships, RDF, Graphql
        *         * Resource URI & URI Endpoints (URL)
