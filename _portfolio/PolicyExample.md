---
title: "Example of a data architecture policy"
excerpt: "Draft?"
collection: portfolio
venue: 'Vision'
tags:
  - Governance
  - Management
---

I have seen many policy documents which list out roles, responsibilities, a list or policy statements and I get a sense that it is an empty document. I had a conversation with someone in corporate policy and she guided me on a very effective format which is both simple and serves the intent of a policy which is to guide decision making.

* Context: background of the policy
* Decision: the agreed guideline to handle the situtation in the context
* Consequence: the ramification of not adhereing to the policy
* Architecture Guidelines: impact on the architetcure

I will present the format with a simple example below.

## Context
* Our organization handles various types of customer data, including sensitive personal information. 
* To comply with data protection regulations and ensure customer trust, we must implement strict data retention policies.

## Decision
* Customer data will be retained for a maximum of seven years from the date of the last transaction. 
* After this period, the data will be securely deleted from all storage systems.

## Consequence
* Failing to adhere to this policy will result in data being kept longer than necessary, potentially leading to non-compliance with data protection regulations, increased storage costs, and higher risk of data breaches. 
* Compliance audits will be conducted regularly, and any discrepancies will result in disciplinary actions and mandatory retraining for responsible employees.

## Architecture Guidelines
* Data identification - All Datasets which contain customer data needs to be classified.
* Storage methods — Detail how and where the all copies will be securely stored.
* Approval for usage - The customer data steward requires all consumption and copies of customer data.
* Retention periods — The Start date and end data of customer data requires to be clearly identified.
* Disposal procedures —  Erasing or destroying data at the end of its lifecycle.
* Data formats — All customer data needs to eb
* Backup & archiving — Establish procedures for creating redundant copies and long-term storage.
* Access — Only approved personal with the right roles and access clearance can have access to customer data.
