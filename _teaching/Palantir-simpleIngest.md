---
title: "Palantir Simple Injest from Postgress"
collection: teaching
type: "Palantir"
permalink: /teaching/Palantir-Simple-PostgressIngest
venue: "Palantir"
date: 2026-06-07
---

# Some intitial thoughts
- We will Use Python (Lightweight) for most ingestion use cases. It's simpler, faster to start up (no Spark overhead), and handles the vast majority of table sizes. This is what we will be building now with @external_systems + psycopg2.
- Use will PySpark only when you need Spark's distributed JDBC reader to parallelize reads across partitions for very large tables. Spark can split a single table read into multiple parallel queries using a partition column: We will try an demonstrate how we would achieve this in a later use case

We developed how to create a data connection


## Code Repository

Code Repository in Foundry

- A Code Repository is a Git-backed project that contains the logic for data transformations, functions, models, or applications. It's the fundamental unit where all "code" lives in Foundry.

- What It Contains

code-repository/
├── transforms-python/
│   ├── conda_recipe/
│   │   └── meta.yaml              ← Dependencies (packages)
│   └── src/
│       ├── myproject/
│       │   ├── pipeline.py         ← Registers transforms for discovery
│       │   └── datasets/
│       │       └── my_transform.py ← Your transform logic
│       ├── setup.py                ← Entry point declaration
│       └── test/                   ← Unit tests
├── build.gradle                    ← Build configuration
└── ci.yml                          ← CI pipeline definition


<img width="612" alt="image" src="nuneskris/nuneskris.github.io/_teaching/images/palantir/image-4.png">


![alt text](image-4.png)

