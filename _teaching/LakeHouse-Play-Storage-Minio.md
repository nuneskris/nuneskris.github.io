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
***1. Install Docker Desktop***
***2. Search and install image: bitnami/minio:latest***
   
   ![image](https://github.com/nuneskris/nuneskris.github.io/assets/82786764/b380c61b-0443-412b-a822-1d981760c069)

   The userid and password are requied to loginto the webui.
   
   ![image](https://github.com/nuneskris/nuneskris.github.io/assets/82786764/eb31cefb-39f6-4b96-a251-30be48380eb9)

# Working with Mino
***1. Open the URL with the user-id and password.***
***2. Create a bucket. We are able to create bucket similar to s3***
   
 ![image](https://github.com/nuneskris/nuneskris.github.io/assets/82786764/1cb79ca4-cf17-435c-83f8-6a26d6d45f59)

# Minio Client API
***1. Install Python Client***

 [Python setup](https://min.io/docs/minio/linux/developers/python/minio-py.html)
   
         pip3 install minio
   
```python

from minio import Minio
from minio.error import S3Error
from dotenv import load_dotenv
import glob    
import os
```

***3. Connect to Minio***
   
         MINIO_CLIENT = Minio("localhost:55003", access_key=ACCESS_KEY, secret_key=SECRET_KEY, secure=False)

Find the Network details from Docker. Open the container, navigate to Inspect and Networks section. Port 55003 on your local machine maps to port 9000 inside the container (Minio's default port for the main service).

   ![image](https://github.com/nuneskris/nuneskris.github.io/assets/82786764/248a331b-cb4a-4151-a5be-428073f1b3de)

we can also use docker cli for this.

      docker ps
      CONTAINER ID   IMAGE                      COMMAND                  CREATED        STATUS             PORTS                                                                                                  NAMES
      f1af05f8d214   bitnami/minio:latest       "/opt/bitnami/script…"   10 hours ago   Up About an hour   0.0.0.0:55003-       >9000/tcp, 0.0.0.0:55002->9001/tcp                                                       minimino

***Python code to put files to the bucket***

```python
load_dotenv()
myfile = '/#######/#######/###/##/#####/DataLakeHouse/FileFormat/Parquet/BestPractices/ProvisionData'
LOCAL_FILE_PATH = os.environ.get(myfile)
# Get the access key from  Minio Web Admin UI
ACCESS_KEY = 'm0iXygQv3PeNyR3KjX9U'
SECRET_KEY ='rmHl5PWmEn3NYYOCHbIZKIISeTogD06UhGQk4ltf'
#MINIO_API_HOST = "http://localhost:9000"
print('Connecting to the Minio Server')
MINIO_CLIENT = Minio("localhost:55003", access_key=ACCESS_KEY, secret_key=SECRET_KEY, secure=False)
BUCKET_NAME = 'lakehouse-storage'

try:  
    found = MINIO_CLIENT.bucket_exists(BUCKET_NAME)
    if not found:
        MINIO_CLIENT.make_bucket(BUCKET_NAME)
    else:
        print("Bucket already exists")
    print("Connected")
    upload_local_directory_to_minio(myfile, BUCKET_NAME, "parquet", MINIO_CLIENT)
    print("It is successfully uploaded to bucket")
except S3Error as e:
        print("Error occurred:", e.code, e.message, e._bucket_name)
# Put only uplaods one file at a time, so this is a way to upload multiple files by parsing through the folders.
def upload_local_directory_to_minio(local_path, bucket_name, minio_path, client):
    assert os.path.isdir(local_path)

    for local_file in glob.glob(local_path + '/**'):
        local_file = local_file.replace(os.sep, "/") # Replace \ with / on Windows
        print(local_file)
        if not os.path.isfile(local_file):
            upload_local_directory_to_minio(
                local_file, bucket_name, minio_path + "/" + os.path.basename(local_file), client)
        else:
            remote_path = os.path.join(
                minio_path, local_file[1 + len(local_path):])
            remote_path = remote_path.replace(
                os.sep, "/")  # Replace \ with / on Windows
            client.fput_object(bucket_name, remote_path, local_file)
```
***Verifying the data is in the bucket***

![image](https://github.com/nuneskris/nuneskris.github.io/assets/82786764/337f902f-c509-40fa-924e-19a816782d68)


