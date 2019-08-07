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
* Multiple security groups can be assigned to a single EC2 instance
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
* Change in security group takes effect immediately
* Security groups are stateful and thus when an inbound rule is created the security group also allows the traffic out
* Access Control Lists are stateles and hence both inbound and outbound rules need to be created
* Security groups cannot blacklist an IP or port. Everything is blocked by default, we need to specifically open ports
* `http://169.254.169.254/latest/user-data/` gives user data scripts
* `http://169.254.169.254/latest/meta-data/` gives meta data
* `http://169.254.169.254/latest/meta-data/public-ipv4/` gives public IP
* `http://169.254.169.254/latest/meta-data/local-ipv4/` gives local IP
* Two types of placement groups
  * Clustered Placement Group - Grouping of instances within a single AZ. Recommended for applications that need low network latency and high network throughput. Only certain instances can be launched in this placement group. It cannot spac multiple AZ
  * Spread Placement Group - Instances are placed in distinct hardware. Recommended for applications that have a small number of critical instances that should be kept separate from each other. It can span multiple AZ
* The instances within a placement group should be homogeneous
* Existing instances can't be moved into a placement group
* Placement groups can't be merged

## EFS

* Supports Network File System version 4 (NFSv4)
* Read after write consistency
* Data is stored across multiple AZ's within a region
* No pre-provisioning required

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
* ELBs do not have a predefined IPv4 address. We resolve to them using a DNS name

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
  * **General Purposse SSD (GP2)** - General purpose SSD volume
  * **Provisioned IOPS SSD (IO1)** - Highest-performance SSD volume for mission-critical low-latency or high throughput workloads. Good for Databases.
  * **Throughput Optimized HDD (ST1)** - Low cost HDD volume designed for frequently accessed, throughput intensive workloads. Good for big data and datawarehouses
  * **Cold HDD (SC1)** - Lowest cost HDD volume designed for less frequently accessed workloads. Good for file servers
  * **EBS Magnetic HDD (Standard)** - Previous generation HDD. For workloads where data is infrequently accessed
* The size and IOPS (only for IO1) can be increased
* Increasing the size of the volume does not automatically increase the size of the partition
* EBS volumes can be backed up using snapshots
* Snapshots are also used to resizing a volume down, changing the volume type and encrypting a volume
* Snapshots occupy only the size of data
* EBS Encryption leverages keys from KMS (AES-256)
* Copying an unencrypted snapshot allows encryption
* All the data in flight moving between the instance and an encrypted volume is encrypted
* EBS backups use IO and hence backups should be taken during off-peak hours
* Each EBS volume is automatically replicated within its own AZ to protect from component failure and provide high availability and durability
* EC2 instance and its volume are going to be in the same AZ
* Migrating EBS to a different AZ or Region
  * Create a snapshot
  * Create an AMI
  * Launch an EC2 instance in a different AZ with the AMI
  * Copy the AMI to a different Region
  * Launch an EC2 instance in a different Region with the AMI
* The size and type of the EBS volumes can be changed without even stopping the EC2 instance
* Snapshots exist on S3
* Snapshots are incremental
* To take a snapshot of the root device, the instance needs to be stopped
* AMI can be created directly from the volume as well
* AMI root device storage can be
  * Instance Store (Ephemeral Stores)
  * EBS Backed Volumes
* Instance store volumes are created from a template stored in Amazon S3
* Instance stores are attached to the host where the EC2 is running, whereas EBS volumes are network volumes. However, in 90% of the use cases the difference in latency with the two types of stores does not make any difference
* Throughput = IOPS * I/O size. The I/O size is 256KB (earlier 16KB). If the IOPS provisioned is 500, the instance can achieve 500 256KB writes per second
* To encrypt a root volume, 
  * Take a snapshot
  * Copy the snapshot and choose the ecryption option (Once encrypted, it cannot be uncrypted by again making a copy)
  * Create an Image (AMI) from the encrypted snapshot
* Snapshots of encrypted volumes are always encrypted
* Volumes restored from encrypted snapshots are encrypted automatically
* Snapshots can be shared, but only if they are unencrypted

## CloudWatch

* CloudWatch is for monitoring performance, whereas CloudTrail is for auditing API calls
* CloudWatch with EC2 will monitor events every 5 min by default. With detailed monitoring, the interval will be 1 min
* CloudWatch alarms can be created to trigger notifications

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
  * Routing policy: **simple**, **failover**, **geolocation**, **geoproximity**, **latency**, **weighted**, **multivalue answer**
* IPv4 - 32 bit, IPv6 - 128 bit
* **Simple Routing** - Multiple IP addresses against a single A record. Route 53 returs all of them in random order
* **Wighted Routing** - A separate A record for each IP with a percentage weight. A separate health check can be associated with each IP or A record. SNS notification can be sent if a health check fails. If a health check fails, the server is removed from Route 53, until the health check passes
* **Latency Based Routing** - A separate A record for each IP with a percentage weight. A separate health check can be associated with each IP or A record. Routing happens to the server with lowest latency
* **Failover Routing** - 2 separate A records - one for primary and one for secondary. Health check can be associated with each, If primary goes down, traffic will all be ruted to secondary
* **Geolocation Based Routing** - A separate A record for each IP. Each A record is mapped to a location and the routing happens to a specific server depending on which location the DNS query originated. Good for scenarios where different website will have different language laels based on location

## RDS

* Upto 5 Read Replicas (Async Replication - within AZ, cross AZ or cross Region)
* Replication for Disaster Recovery is synchronous (across AZ - Automatic failover - DNS endpoint remains same) - Multi AZ
* Replicas can be promoted to their own DB
* Automated backups:
  * Daily full snapshot of the database
  * Capture transaction logs in real time
  * Ability to restore to any point in time
  * 7 days retention (can be increased to 35 days)
* DB Snapshots:
  * Manually triggered by the user
  * Retention of backup for as long as we want
* Encryption at rest capability with AWS KMS - AES-256 encryption
* To enforce SSL:
  * PostgreSQL: rds.force_ssl=1 in the AWS RDS Console (Paratemer Groups)
  * MySQL: Within the DB: GRANT USAGE ON *.* TO 'mysqluser'@'%' REQUIRE SSL;
* To connect using SSL:
  * Provide the SSL Trust certificate (can be download from AWS)
  * Provide SSL options when connecting to database
* RDS, in general, is not serverless (except Aurora Serverless which is serverless)
* We cannot access the RDS virtual machines. Patching the RDS operating system is Amazon's responsibility
* The backup data is stored in S3
* Backups are taken during specified window. The application may experience elavated latency during backup
* Restoring DB from automatic backup or snapshots always creates a new RDS instance with a new DNS endpoint
* Read replicas of read replicas are possible
* Each read replica will have its own DNS endpoint
* Read replica can be created in a separate region as well
* If a read replica is promoted to its own database, the replication will stop
* Read replica cannot be enabled unless the automatic backups are also enabled
* Read replicas themselves can be Multi-AZ for disaster recovery
* A failover in a Multi-AZ deployment can be forced by rebooting the DB
* Two ways of improving performance
  * Read replicas
  * ElasticCache

## VPC

* VPC Flow Logs allows us to monitor the traffic within, in and out of your VPC (useful for security, performance, audit)
* VPC are per Account per Region
* Subnets are per VPC per AZ
* Elastic cache can be used as - db cache, session store
* Caching patterns - 
  * Write through
  * Lazy loading

## DynamoDB

* Supports both document and key-value data model
* Stored on SSD storage
* Spread across 3 geographically distributed data centers
* Supports both Eventual Consistant Reads (Default) & Strongly Consistant Reads
* Serverless service

## Redshift

* Amazon's data warehouse solution
* Single node (160 GB) or multi node (leader node and compute node - upto 128 compute nodes)
* Column based data store, column based compression techniques and multiple other compression techniques
* No indexes or materialized views
* Massively parallel processing
* Redshift attempts to maintain 3 copies of data (the original and replica on the compute nodes and a backup in S3)
* Available in only 1 AZ
* Backup retention period is 1 day by default which can be extended to 35 days
* Can asynchronously replicate to S3 in a different region for disaster recovery

## Aurora

* Aurora storage automatically grows in increments of 10GB, up to 64 TB
* Aurora can have 15 replicas while MySQL has 5, and the replication process is faster (sub 10 ms replica lag)
* 2 copies of data is maintained in each AZ with a minimum of 3 AZ
* Copute resources can scale upto 32 vCPUs and 244 GB of memory
* Aurora can transparently handle the loss of 2 copies of data without affecting write availability and 3 copies of data without affecting read availability
* Backups and snapshots does not impact database performance
* Storage is self-healing. Disks and blocks are scanned for errors and repaired automatically
* Aurora snapshots can be shared with other AWS accounts
* Two types of replicas - MySQL replicas and Aurora Replicas
* Automated failover is only possible with Aurora replicas
* Failover in Aurora is instantaneous. It’s HA native

## ElasticCache

* ElastiCache is to get managed Redis or Memcached
* ElasticCache features - 
  * Write Scaling using sharding
  * Read Scaling using Read Replicas
  * Multi AZ with Failover Capability
* Redis - Multi AZ, Backups and restore
* Memcached - Multi threaded, horizontal scaling

