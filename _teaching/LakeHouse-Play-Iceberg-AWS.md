---
title: "Iceberg on AWS - Hello World"
collection: teaching
type: "Lakehouse"
permalink: /teaching/LakeHouse-Play-Iceberg-AWS
date: 2024-06-01
venue: "Iceberg"
date: 2024-06-01
location: "AWS"
---

# Setup
### 1. AWS Glue can create Iceberg Tables directily
NOTE: I was suprised to find out AWS considers Iceberg as a first class table format. They must be seing promise with the tech. 
![image](https://github.com/user-attachments/assets/c89b44a7-d8bc-4b34-b78b-d2dae3a62c6e)

### 2. Setting up a schema
```json
[
  {
    "Name": "NAME_FIRST",
    "Type": "string",
    "Comment": ""
  },
  {
    "Name": "NAME_MIDDLE",
    "Type": "string",
    "Comment": ""
  },
  {
    "Name": "NAME_FIRST",
    "Type": "string",
    "Comment": ""
  },
  {
    "Name": "NAME_LAST",
    "Type": "string",
    "Comment": ""
  },
  {
    "Name": "NAME_INITIALS",
    "Type": "string",
    "Comment": ""
  },
  {
    "Name": "SEX",
    "Type": "string",
    "Comment": ""
  },
  {
    "Name": "LANGUAGE",
    "Type": "string",
    "Comment": ""
  },
  {
    "Name": "PHONENUMBER",
    "Type": "string",
    "Comment": ""
  },
  {
    "Name": "EMAILADDRESS",
    "Type": "string",
    "Comment": ""
  },
  {
    "Name": "LOGINNAME",
    "Type": "string",
    "Comment": ""
  },
  {
    "Name": "ADDRESSID",
    "Type": "int",
    "Comment": ""
  },
  {
    "Name": "VALIDITY_STARTDATE",
    "Type": "int",
    "Comment": ""
  },
  {
    "Name": "VALIDITY_ENDDATE",
    "Type": "int",
    "Comment": ""
  }
]
```
