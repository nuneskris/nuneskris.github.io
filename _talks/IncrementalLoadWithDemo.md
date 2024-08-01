---
title: "Incremental Load with Demo"
collection: talks
type: "Data Modeling"
permalink: /talks/IncrementalLoadWithDemo
date: 2024-03-01
venue: "SCD2"
date: 2024-06-01
location: "AWS"
---

# Setup
Updating the date audit columns to timestamp for the purpose of this demo. Used excel formula for preping the data
```
=TEXT(DATE(LEFT(A1, 4), MID(A1, 5, 2), RIGHT(A1, 2)) + TIME(12, 0, 0), "yyyy-mm-dd HH:MM:SS") & ".000"
```
![image](https://github.com/user-attachments/assets/741fff01-aac9-47ae-861f-b03fc9ce80b5)

