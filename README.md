# AWS Cheat Sheet (As of Aug, 2019)

## S3

* **Object** based storage (files)
* Files can be from **0 Bytes & 5 TB**
* Bucket web address: `https://s3-<AZ name>.amazonaws.com/<bucketname>` e.g. `https://s3-eu-west-1.amazonaws.com/myuniquename`
* Bucket name has to be **unique** across **all regions**
* **Read after write** consistency for **PUTs of new objects**
* **Eventual** consistency for **overwrite PUTs** and **DELETEs**
* **Designed** for **99.99%** availability for S3 Standard, **99.9%** for S3 - IA & **99.5%** for S3 One Zone - IA
* Amazon **guarantees** 99.9% availability for S3 Standard, **99%** for S3 - IA & S3 One Zone - IA
* Amazon **guarantees 99.999999999% (11 * 9's) durability** for all storage classes
* Provides **Multifactor Authentication** for **deletes**
* **S3 Standard** - Replicated to **>= 3 AZ** (except S3 One Zone IA)
* **S3 Infrequently Accessed (IA)** - Provides **rapid access** when needed
* **S3 Infrequently Accessed (IA)** - **Less storage cost** but has **data retrieval cost**
* **S3 One Zone IA** - Data is stored in a **single AZ** + **Retrieval charge**
* **S3 Intelligent Tiering** - **ML based** - moves object to different storage classes based on its learning about usage of the object
* **S3 Glacier** - **Archiva** + **Retrival time configurable** from minutes to hours + **Retrieval charge**
* **S3 Glacier Deep Archive** - Retrieval time of **12 hrs** + **Retrieval charge**
* Charged based on 
  * Storage
  * No. of Requests
  * Storage Management (Tiers)
  * Data Transfer
  * Transfer Acceleration
  * Cross Region Replication
* Cross Region Replication for High Availability or Disaster Recovery
* Transfer Acceleration for reduced upload time
* Transfer Acceleration takes advantage of Cloudfronts globally distributed edge locations and then routes data to the S3 bucket through Amazon's internal backbone network
* The bucket access logs can be stored in another bucket owned by the **same AWS account** in the **same region** 
* Enabling logging on a bucket from console also updates the ACL on the target bucket to grant write permission to the Log Delivery group.
* Encryption at Rest - 
  * SSE-S3 (Amazon manages key)
  * SSE-KMS (Amazon + Us manage keys)
  * SSE-C (Us manage keys)
  * Client Side Encryption (Us manage keys and encrypt objects)
* Versioning, once enabled, cannot be disabled. It can only be suspended
* Lifecycle management rules (movement to different storage type and expiration) can be set for current version and previous versions separately
* Cross region replication requires versioning to be enabled in both source and destination
* Cross region replication not going to replicate
  * file versions created before enabling cross region replication
  * delete marker
  * version deletions
* Files are stored as key-value pair. Key is the file name with the entire path and value is the file content as a sequence of bytes
* More than 5 GB files must be uploaded using multipart upload
* 