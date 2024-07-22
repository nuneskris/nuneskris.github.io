---
title: "What is unified data"
excerpt: "multiple sources of the same data <br/><img src='/images/portfolio/pub_vision_dream.png'>"
collection: portfolio
date: 2024-01-03
venue: 'Vision'
---

# The Same Data Is Defined Inconsistently Across Multiple Systems

Different systems are designed and built in isolation, storing data in native data model schemas and formats. A key challenge to integrating data is to relate data across these silos. Very often, the same data is defined inconsistently across multiple systems. Let's take customer data as an example.

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
