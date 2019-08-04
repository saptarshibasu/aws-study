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

## IAM

* Global service
* Users for individuals
* User Groups for grouping users with similar permission requirements
* Roles are for machines or internal AWS resources. One IAM Role for ONE application

## EC2

* Security Groups are for network security
* Security groups are locked down to a region / VPC combination
* One security group can be attached to multiple EC2 instances
* If connection to application times out, it could be a security group issue
* If connection is refused, it's an application issue
* All inbound traffic is blocked by default
* all outbound traffic is authorized by default
* Security Groups can refer to other Security Groups
* Elastic IP gives a fixed IP to an EC2 instance across restarts
* By default one AWS account can have 5 elastic IP
* Prefer load balancer over Elastic IP
* EC2 User Data script runs once (with root privileges) at the instance first start
* EC2 Launch types - 
  * **On-demand instances** - short workload, predictable pricing
  * **Reserved instances** - long workload (>= 1 year)
  * **Convertible reserved instances** - long workload with flexible instance types
  * **Scheduled reserved instances** - reserved for specific time window
  * **Spot instances** - short workload, cheap, can lose instances, good for batch jobs, big data analytics etc.
  * **Dedicated instances** - no other customer will share the hardware, but instances from same AWS account can share hardware, no control on instance placement
  * **Dedicated hosts** - the entire server is reserved, provides more control on instance placement, more visibility into sockets and cores, good for "bring your own licenses", complicated regulatory needs
* Billing by second with a minimum of 60 seconds
* A custom AMI can be created with pre-installed software packages, security patches etc. instead of writing user data scripts, so that the boot time is less during autoscaling
* AMIs are built for a specific region, but can be copied across regions
* T2/T3 are burstable instances. Spikes are handled using burst credits that are accumulated over time. If burst credits are all consumed, performance will suffer
* M instance types are balanced
  
## ELB
  
* **ELB types** - 
  * Classic Load Balancer (V1 - old generation)
  * Application Load Balancer (V2 - new generation) - Layer 7
  * Network Load Balancer (V2 - new generation) - Layer 4
* ELB provides **health check** for instances
* **ALB** can handle **multiple applications** where each application has a traget group and load for that application is balanced across instances within the particular target group
* ALB supports **HTTP/HTTPS** & **Websocket** protocols
* ALB - True IP, port and protocol details of the client are inserted in HTTP headers - **X-Forwarded-For**, **X-Forwarded-Port** and **X-Forwarded-Proto** respectively
* ALB can route based on hostname in URL and route in URL
* Network Load Balancers are mostly used for extreme performance and should not be the default load balancer
* Network Load Balancers have less latency ~100 ms (vs 400 ms for ALB)
* Load Balancers have static host name. DO NOT resolve & use underlying IP
* LBs can scale but not instantaneously – contact AWS for a “warm-up”

## Auto Scaling Group

* Auto scaling group is configured to register new instances to a traget group of ELB
* IAM role attached to the ASG will get assigned to the instances
* If instance gets terminated, ASG will restart it
* If instance is marked as unhealthy by load balancer, ASG will restart it
* ASG can scale based on CloudWatch alarms
* ASG can scale based on custom metric sent by applications to CloudWatch

## EBS

* An EBS volume is a **network drive**
* An EC2 machine by default loses its root volume when terminated
* It's locked to an AZ. To move a volume to a different AZ, a snapshot needs to be created
* EBS Volume types:
  * **GP2(SSD)** - General purpose SSD volume
  * **IO1(SSD)** - Highest-performance SSD volume for mission-critical low-latency or high throughput workloads
  * **ST1(HDD)** - Low cost HDD volume designed for frequently accessed, throughput intensive workloads
  * **SC1(HDD)** - Lowest cost HDD volume designed for less frequently accessed workloads
* The size and IOPS (only for IO1) can be increased
* Increasing the size of the volume does not automatically increase the size of the partition
* EBS volumes can be backed up using snapshots
* Snapshots are also used to resizing a volume down, changing the volume type and encrypting a volume
* Snapshots occupy only the size of data
* EBS Encryption leverages keys from KMS (AES-256)
* Copying an unencrypted snapshot allows encryption
* All the data in flight moving between the instance and an encrypted volume is encrypted
* EBS backups use IO and hence backups should be taken during off-peak hours

## Route 53
  
* In AWS, the most common records are:
  * **A**: URL to IPv4
  * **AAAA**: URL to IPv6
  * **CNAME**: URL to URL
  * **Alias**: URL to AWS resource.
* Prefer Alias over CNAME for AWS resources (for performance reasons)
* Route53 has advanced features such as:
  * Load balancing (through DNS – also called client load balancing)
  * Health checks (although limited…)
  * Routing policy: **simple**, **failover**, **geolocation**, **geoproximity**, **latency**, **weighted**

## RDS

* Upto 5 Read Replicas (Async Replication - within AZ, cross AZ or cross Region)
* Replication for Disaster Recovery is synchronous (across AZ - Automatic failover)
* Replicas can be promoted to their own DB
* Automated backups:
  * Daily full snapshot of the database
  * Capture transaction logs in real time
  * Ability to restore to any point in time
  * 7 days retention (can be increased to 35 days)
* DB Snapshots:
  * Manually triggered by the user
  * Retention of backup for as long as you want
* Encryption at rest capability with AWS KMS - AES-256 encryption
* To enforce SSL:
  * PostgreSQL: rds.force_ssl=1 in the AWS RDS Console (Paratemer Groups)
  * MySQL: Within the DB: GRANT USAGE ON *.* TO 'mysqluser'@'%' REQUIRE SSL;
* To connect using SSL:
  * Provide the SSL Trust certificate (can be download from AWS)
  * Provide SSL options when connecting to database
* Aurora storage automatically grows in increments of 10GB, up to 64 TB
* Aurora can have 15 replicas while MySQL has 5, and the replication process is faster (sub 10 ms replica lag)
* Failover in Aurora is instantaneous. It’s HA native
* ElastiCache is to get managed Redis or Memcached
* ElasticCache features - 
  * Write Scaling using sharding
  * Read Scaling using Read Replicas
  * Multi AZ with Failover Capability

## VPC

* VPC Flow Logs allows us to monitor the traffic within, in and out of your VPC (useful for security, performance, audit)
* VPC are per Account per Region
* Subnets are per VPC per AZ
* Elastic cache can be used as - db cache, session store
* Caching patterns - 
  * Write through
  * Lazy loading