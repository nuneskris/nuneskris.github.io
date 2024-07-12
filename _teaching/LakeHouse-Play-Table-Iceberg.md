---
title: "MinIO Object Storage for Linux Locally (anywhere)"
collection: teaching
type: "Lakehouse"
permalink: /teaching/LakeHouse-Play-Table-Iceberg
date: 2024-07-01
venue: "Iceberg, Spark"
date: 2024-06-01
location: "Docker"
---

It took a while trying to get Iceberg to work. I had to deal with way to many jar dependencies. I tried and tried and it was taking more time than I planned and had over a Saturday.
So I did take the easy way out by reusing images which were packaged and ready to go. Follow along the post by [Alex Merced](https://alexmercedcoder.medium.com/creating-a-local-data-lakehouse-using-spark-minio-dremio-nessie-9a92e320b5b3) for the docker set up.

I used Docker Desktop, becaue it is easy to use and I do not have to recall how to use it.

The docker-compose.yml uses the dremio image which has all the Spark related dependencies already packaged. I will try to build one in the future when I have the time. Minio (Which I love) for storage, Nessie for Catalog services and the Sparknotebook.

    services:
      dremio:
        platform: linux/x86_64
        image: dremio/dremio-oss:latest
        ports:
          - 9047:9047
          - 31010:31010
          - 32010:32010
        container_name: dremio
      minioserver:
        image: minio/minio
        ports:
          - 9000:9000
          - 9001:9001
        environment:
          MINIO_ROOT_USER: minioadmin
          MINIO_ROOT_PASSWORD: minioadmin
        container_name: minio
        command: server /data --console-address ":9001"
      spark_notebook:
        image: alexmerced/spark33-notebook
        ports:
          - 8888:8888
        volumes:
          - ./data:/data  # Mounting the host directory ./data to /data in the container
        env_file: .env
        container_name: notebook
      
      nessie:
        image: projectnessie/nessie
        container_name: nessie
        ports:
          - "19120:19120"
    networks:
      default:
        name: iceberg_env
        driver: bridge


First we would need to setup a bucket in Minio with Access Keys which are used by Spark to integrate with Minio. Its is an empty bucket.

![image](https://github.com/user-attachments/assets/ddb94b8d-4f3c-4664-b591-f5ca00e6f51b)

