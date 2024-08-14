---
title: "Example of a data architecture policy"
excerpt: "A more effective format that is both simple and truly serves the intent of a policy"
collection: portfolio
venue: 'Vision'
tags:
  - Governance
  - Management
---

I've encountered many policy documents that list roles, responsibilities, and policy statements but often feel empty and ineffective. In a conversation with someone in corporate policy, I was guided towards a more effective format that is both simple and truly serves the intent of a policy: guiding decision-making.

# Effective Policy Format:

1. Context: Background of the policy.
2. Decision: The agreed guideline to handle the situation in the context.
3. Consequence: The ramifications of not adhering to the policy.
4. Architecture Guidelines: The impact on the architecture.

I'll present this format with a simple example below.

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

1. ***Data Identification***: All datasets containing customer data must be classified.
2. ***Storage Methods***: Detail how and where all copies will be securely stored.
3. ***Approval for Usage***: The customer data steward must approve all consumption and copies of customer data.
4. ***Retention Periods***: The start and end dates of customer data must be clearly identified.
5. ***Disposal Procedures***: Procedures for erasing or destroying data at the end of its lifecycle.
6. ***Data Formats***: All customer data needs to be stored in consistent and approved formats.
7. ***Backup & Archiving***: Establish procedures for creating redundant copies and long-term storage.
8. ***Access***: Only approved personnel with the right roles and access clearance can access customer data.
