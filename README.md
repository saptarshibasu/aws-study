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
* Replicated to **>= 3 AZ** (except S3 One Zone IA)
* **S3 Standard** - Frequently accessed
* **S3 Infrequently Accessed (IA)** - Provides **rapid access** when needed
* **S3 Infrequently Accessed (IA)** - **Less storage cost** but has **data retrieval cost**
* **S3 One Zone IA** - Data is stored in a **single AZ** + **Retrieval charge**
* **S3 Intelligent Tiering** - **ML based** - moves object to different storage classes based on its learning about usage of the object
* **S3 Glacier** - **Archiva** + **Retrival time configurable** from minutes to hours + **Retrieval charge**
* **S3 Glacier Deep Archive** - Retrieval time of **12 hrs** + **Retrieval charge**
* **S3 Reduced Redundacy** - Deprecated. Sustains loss of data in a single facility
* Charged based on 
  * Storage
  * No. of Requests
  * Storage Management (Tiers)
  * Data Transfer
  * Transfer Acceleration
  * Cross Region Replication
* Cross Region Replication for High Availability or Disaster Recovery
* Transfer Acceleration for reduced upload time
* Transfer Acceleration takes advantage of Cloudfront's globally distributed edge locations and then routes data to the S3 bucket through Amazon's internal backbone network
* The bucket access logs can be stored in another bucket owned by the **same AWS account** in the **same region** 
* Enabling logging on a bucket from console also updates the ACL on the target bucket to grant write permission to the Log Delivery group.
* **Encryption at Rest** - 
  * SSE-S3 (Amazon manages key)
  * SSE-KMS (Amazon + Us manage keys)
  * SSE-C (Us manage keys)
  * Client Side Encryption (Us manage keys and encrypt objects)
* **SSE-S3**
  * Server side encryption
  * Key managed by S3
  * AES 256
  * Must set header - `"x-amz-server-side-encryption":"AES256"`
* **SSE-KMS**
  * Server side encryption
  * Key managed by KMS
  * More control on rotation of key
  * Audit trail on how the key is used
  * Must set header - `"x-amz-server-side-encryption":"aws:kms"`
* **SSE-C**
  * Server side encryption
  * Key managed by customer outside AWS
  * HTTPS is a must
  * Key to supplied in HTTP header of every request
* Client Side Encryption
  * Key managed by customer outside AWS
  * Clients encrypt / decrypt data
* Versioning, once enabled, cannot be disabled. It can only be suspended
* Lifecycle management rules (movement to different storage type and expiration) can be set for current version and previous versions separately
* Lifecycle management rules can be used to delete incomplete multipart uploads after a configurable no. of days
* Cross region replication requires versioning to be enabled in both source and destination
* Cross region replication not going to replicate
  * file versions created before enabling cross region replication
  * delete marker
  * version deletions
* Files are stored as key-value pair. Key is the file name with the entire path and value is the file content as a sequence of bytes
* More than **5 GB** files must be uploaded using **multipart upload**
* Files larger than **100 MB** should be uploaded using **multipart upload**
* Multipart upload -
  * Retry is faster
  * Run in parallel to improve performance and utilize network bandwidth
* There can be at most 10 tags
* Versioning is enabled at the bucket level
* **S3 Bucket Policy** can be used to enforce upload of only encrypted objects. The policy will deny any PUT request that does not have the appropriate header
* S3 static website URL: `<bucket-name>.s3-website-<aws-region>.amazonaws.com` or `<bucket-name>.s3-website.<aws-region>.amazonaws.com`
* S3 Bucket Policy can be used to provide public read access to all files in the bucket instead of providing public access to each individual file
* When each individual file has public access, uploading a new version of an existing file doesn't automatically give public access to the latest version (unless an S3 bucket policy exists to give public access)
* When encryption is enabled on an existing file, a new version will be created (provided versioning is enabled)
* In a bucket with versioning enabled, even if the file is deleted, the bucket cannot be deleted using AWS CLI until all the versions are deleted
* S3 evaluates and applies bucket policies before applying bucket encryption settings. Even if bucket encryption settings is enabled, PUT requests without encryption information will be rejected if there are bucket policies to reject such PUT requests
* Prefer default encryption settings over S3 bucket policies to encrpt objects at rest
* If versioning is enabled, S3 can be configured to require **multifactor authentication** for 
  * permanently deleting an object version
  * suspend versioning on bucket
* Only root account can enable **MFA Delete** using CLI
* **Cross region replication** is asynchronous
* **Cross region replication** can replicate to buckets in different account
* **Pre-signed URL** allows users to get temporary access to buckets and objects
* **S3 Inventory** allow producing reports about S3 objects daily or weekly in a different S3 bucket
* **S3 Inventory** report format can be specified and the data can be queries using Athena
* Storage class needs to be specified during object upload
* **S3 Analytics**, when enabled, generates reports in a different S3 bucket to give insights about the object usage and this can be used to recommend when the object should be moved from one storage class to another
* Glacier - **Retrieval Policy**
  * Expedited (1 - 5 mins retrieval)
  * Standard (3 - 5 hours)
  * Bulk (5 - 12 hours)
* Glacier - object == **archive** (upto 40 TB)
* Glacier - bucket == **vault**
* Glacier - Each vault has ONE vault policy & ONE lock policy
* **Vault policy** - similar to bucket policy - restrict user access
* **Lock policy** - immutable - once set cannot be changed
  * **WORM policy** - write once read many
  * **Forbid deleting** an archive if it is less than 1 year
* Files retrieved from Glacier will be stored in Reduced Redundancy Storage class for a specified number of days
* For faster retrieval based on Retrieval Policy, Capacity Units may need to be purchased
* Earlier S3 performance would start degrading with 100 TPS
* Historically the recommended approach is to have random 4 characters in front of the key name for better distribution of objects across partitions
* With SSE-KMS enabled, the KMS limits might need to be increased to avoid throtling of lot of small uploads
* S3 & Glacier Select - 
  * Allows to select a subset of rows and colums using SQL without retrieving the entire file
  * Joins and subqueries not allowed
  * Files can be compressed with GZIP or BZIP2
  * Works with file format CSV, JSON, Parquet

## Cloudfront

* Content Delivery Network (CDN)
* Caches content at edge location
* Popular with S3, but also works with EC2 & Load Balancer
* Supports RTMP (media) protocol
* Reduced latency, reduced load on server
* Origin Access Identity (OAI) is an user used by Cloudfront to access the S3 files
* S3 bucket policy gives access to OAI and thus preventing users from directly accessing the S3 files bypassing the Cloudfront
* Cloudfront accesslogs can be stored in an S3 bucket

## Snowball

* If it takes more than a week to transfer data over the network, prefer Snowball
* Snowball Edges have computational capabilities
  * <= 100 TB
  * Can be **Storage Optimized (24 vCPU)** or **Compute Optimized (52 vCPU)** & optional GPU
  * Allows processing on the go
  * Useful for IoT capture, machine learning, data migration, image collation etc.

## Snowmobile

* 100 PB in capacity
* Better than snowball if data to be transferred is more than 10 PB

## Storage Gateway

* Storage Gateway supports hybrid cloud by allowing the on-primise resources access the cloud storage like EBS, S3 etc. through standard protocols
* Storage Gateway types - 
  * File Gateway
* File Gateway - 
  * Configured S3 buckets accessible through NFS & SMB protocol
  * Bucket access using IAM roles for each File Gateway
  * Most recently used data is cached in File Gateway
  * Can be mounted on many servers
* Volume Gateway - 
  * Block storage using iSCSI protocol backed by S3
  * EBS snapshots stored S3 buckets
  * Cached Volume - Recently used data cached
  * Stored Volume - Entire dataset in premise with scheduled backups in S3
* Tape Gateway - 
  * Backup from on premise tape to S3 Glacier using iSCSI protocol

## Athena

* Serverless service
* Allows to do analytics directly on S3
* Supports data formats CSV, JSON, ORC, Avro, Parquet (built on Presto)
* Uses SQL Language to query the files
* Good for analyzing VPC Flow Logs, ELB Logs etc.