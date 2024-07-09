---
title: "MinIO Object Storage for Linux Locally (anywhere)"
collection: teaching
type: "Lakehouse"
permalink: /teaching/LakeHouse-Play-Storage-Minio
date: 2024-06-01
venue: "Minio"
date: 2024-06-01
location: "Docker"
---

When I play with new technologies, I like to plat it on my machine locally. Minio is a perfect simulation of cloud storage locally. You can deploy it locally and interact it like a S3 object storage.

> MinIO is an object storage solution that provides an Amazon Web Services S3-compatible API and supports all core S3 features. MinIO is built to deploy anywhere - public or private cloud, baremetal infrastructure, orchestrated environments, and edge infrastructure - [From Minio](https://min.io/docs/minio/linux/index.html)

# Installation
1. Install Docker Desktop
2. Search and install image: bitnami/minio:latest
   
   ![image](https://github.com/nuneskris/nuneskris.github.io/assets/82786764/b380c61b-0443-412b-a822-1d981760c069)

   The userid and password are requied to loginto the webui.
   
   ![image](https://github.com/nuneskris/nuneskris.github.io/assets/82786764/eb31cefb-39f6-4b96-a251-30be48380eb9)

# Working with Mino
1. Open the URL with the user-id and password.
2. Create a bucket. We are able to create bucket similar to s3
   
 ![image](https://github.com/nuneskris/nuneskris.github.io/assets/82786764/1cb79ca4-cf17-435c-83f8-6a26d6d45f59)

# Minio Client API
1. [Python setup](https://min.io/docs/minio/linux/developers/python/minio-py.html)
   
         pip3 install minio
   
```python

from minio import Minio
from minio.error import S3Error
from dotenv import load_dotenv
import glob    
import os
```

   







Heading 3
======
