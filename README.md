# AWS Cheat Sheet

## S3

* **Object** based storage (files)
* Files can be from **0 Bytes & 5 TB**
* Bucket web address: `https://s3-<AZ name>.amazonaws.com/<bucketname>` e.g. `https://s3-eu-west-1.amazonaws.com/myuniquename`
* Bucket name has to be **unique** across all regions
* **Read after write** consistency for PUTs of new objects
* **Eventual** consistency for overwrite PUTs and DELETEs
* **Designed for** 99.99% availability for S3 Standard, 99.9% for S3 - IA & 99.5% for S3 One Zone - IA
* Amazon **guarantees availability** - 99.9% for S3 Standard, 99% for S3 - IA & S3 One Zone - IA
* Amazon **guarantees durability** 99.999999999% (11 * 9's) for all storage classes
* **Replicated** to >= 3 AZ (except S3 One Zone IA)
* **S3 Standard** - Frequently accessed
* **S3 Infrequently Accessed (IA)** - Provides rapid access when needed
* **S3 Infrequently Accessed (IA)** - Less storage cost but has data retrieval cost
* **S3 One Zone IA** - Data is stored in a single AZ + Retrieval charge
* **S3 Intelligent Tiering** - ML based - moves object to different storage classes based on its learning about usage of the object
* **S3 Glacier** - Archive + Retrival time configurable from minutes to hours + Retrieval charge
* **S3 Glacier Deep Archive** - Retrieval time of 12 hrs + Retrieval charge
* **S3 Reduced Redundacy** - Deprecated. Sustains loss of data in a single facility
* Charged based on 
  * Storage
  * No. of Requests
  * Storage Management (Tiers)
  * Data Transfer
  * Transfer Acceleration
  * Cross Region Replication
* **Cross Region Replication** for High Availability or Disaster Recovery
* **Cross Region Replication** requires versioning to be enabled in both source and destination
* **Cross Region Replication** not going to replicate
  * file versions created before enabling cross region replication
  * delete marker
  * version deletions
* **Cross Region Replication** is asynchronous
* **Cross Region Replication** can replicate to buckets in different account
* **Transfer Acceleration** for reduced upload time
* **Transfer Acceleration** takes advantage of Cloudfront's globally distributed edge locations and then routes data to the S3 bucket through Amazon's internal backbone network
* The **bucket access logs** can be stored in another bucket owned by the same AWS account in the same region
* Enabling **logging** on a bucket from console also updates the ACL on the target bucket to grant write permission to the Log Delivery group.
* **Encryption at Rest** - 
  * SSE-S3 (Amazon manages key)
  * SSE-KMS (Amazon + User manages keys)
  * SSE-C (User manages keys)
  * Client Side Encryption (User manages keys and encrypt objects)
* **SSE-S3**
  * Server side encryption
  * Key managed by S3
  * AES 256
  * Must set header - **`"x-amz-server-side-encryption":"AES256"`**
* **SSE-KMS**
  * Server side encryption
  * Key managed by KMS
  * More control on rotation of key
  * Audit trail on how the key is used
  * Must set header - **`"x-amz-server-side-encryption":"aws:kms"`**
* **SSE-C**
  * Server side encryption
  * Key managed by customer outside AWS
  * HTTPS is a must
  * Key to supplied in HTTP header of every request
* **Client Side Encryption**
  * Key managed by customer outside AWS
  * Clients encrypt / decrypt data
* When **encryption** is enabled on an existing file, a new version will be created (provided versioning is enabled)
* Prefer default **encryption** settings over S3 bucket policies to encrypt objects at rest
* With **SSE-KMS** enabled, the KMS limits might need to be increased to avoid throttling of lot of small uploads
* **Versioning**, once enabled, cannot be disabled. It can only be suspended
* **Versioning** is enabled at the bucket level
* If **versioning** is enabled, S3 can be configured to require multifactor authentication for 
  * permanently deleting an object version
  * suspend versioning on bucket
* In a bucket with **versioning** enabled, even if the file is deleted, the bucket cannot be deleted using AWS CLI until all the versions are deleted
* When each individual file has public access, uploading a new version of an existing file doesn't automatically give **public access** to the latest version (unless an S3 bucket policy exists to give public access)
* **Lifecycle management rules** (movement to different storage type and expiration) can be set for current version and previous versions separately
* **Lifecycle management rules** can be used to delete incomplete multipart uploads after a configurable no. of days
* Files are stored as **key-value pair**. Key is the file name with the entire path and value is the file content as a sequence of bytes
* More than 5 GB files must be uploaded using **multipart upload**
* Files larger than 100 MB should be uploaded using **multipart upload**
* **Multipart upload** advantages -
  * Retry is faster
  * Run in parallel to improve performance and utilize network bandwidth
* There can be at most 10 tags
* S3 **Bucket Policy** can be used to enforce upload of only encrypted objects. The policy will deny any PUT request that does not have the appropriate header
* S3 **Bucket Policy** can be used to provide public read access to all files in the bucket instead of providing public access to each individual file
* S3 evaluates and applies **bucket policies** before applying bucket encryption settings. Even if bucket encryption settings is enabled, PUT requests without encryption information will be rejected if there are bucket policies to reject such PUT requests
* S3 static website URL: `<bucket-name>.s3-website-<aws-region>.amazonaws.com` or `<bucket-name>.s3-website.<aws-region>.amazonaws.com`
* Only root account can enable **MFA Delete** using CLI
* **Pre-signed URL** allows users to get temporary access to buckets and objects
* **S3 Inventory** allows producing reports about S3 objects daily or weekly in a different S3 bucket
* **S3 Inventory** reports format can be specified and the data can be queries using Athena
* **Storage class** needs to be specified during object upload
* **S3 Analytics**, when enabled, generates reports in a different S3 bucket to give insights about the object usage and this can be used to recommend when the object should be moved from one storage class to another
* **Glacier** - Retrieval Policy
  * Expedited (1 - 5 mins retrieval)
  * Standard (3 - 5 hours)
  * Bulk (5 - 12 hours)
* **Glacier** - object == archive (upto 40 TB)
* **Glacier** - bucket == vault
* **Glacier** - Each vault has ONE vault policy & ONE lock policy
  * **Vault Policy** - similar to bucket policy - restrict user access
  * **Lock Policy** - immutable - once set cannot be changed
    * WORM Policy - write once read many
    * Forbid deleting an archive if it is less than 1 year
* Earlier S3 **performance** would start degrading with 100 TPS
* Files retrieved from **Glacier** will be stored in Reduced Redundancy Storage class for a specified number of days
* For faster retrieval from **Glacier** based on Retrieval Policy, Capacity Units may need to be purchased
* Historically the recommended approach is to have **random 4 characters** in front of the key name for better distribution of objects across partitions
* **S3 & Glacier Select** - 
  * Allows to select a subset of rows and colums using SQL without retrieving the entire file
  * Joins and subqueries not allowed
  * Files can be compressed with GZIP or BZIP2
  * Works with file format CSV, JSON, Parquet

## Cloudfront

* Content Delivery Network (CDN)
* **Cache content** at edge location
* Popular with S3, but also works with **EC2 & Load Balancer**
* Supports **RTMP (media)** protocol
* **Reduced latency** and reduced load on server
* **Origin Access Identity (OAI)** is an user used by Cloudfront to access the S3 files
* **S3 bucket policy** gives access to OAI and thus preventing users from directly accessing the S3 files bypassing the Cloudfront
* Cloudfront **accesslogs** can be stored in an S3 bucket
* To serve the media of a WordPress website from Cloudfront use the Apache .htaccess file to rewrite the URLs

## Snowball

* If it takes **more than a week** to transfer data over the network, prefer Snowball
* **Snowball Edges** have computational capabilities
  * <= 100 TB
  * Can be **Storage Optimized (24 vCPU)** or **Compute Optimized (52 vCPU) & optional GPU**
  * Allows processing **on the go**
  * Useful for IoT capture, machine learning, data migration, image collation etc.

## Snowmobile

* 100 PB in capacity
* Better than snowball if data to be transferred is more than 10 PB

## Storage Gateway

* Storage Gateway supports **hybrid cloud** by allowing the on-primise resources access the cloud storage like EBS, S3 etc. through standard protocols
* Storage Gateway **types** - 
  * File Gateway
  * Volume Gateway
  * Tape Gateway
* **File Gateway** - 
  * Configured S3 buckets accessible through **NFS & SMB** protocol
  * Bucket access using IAM roles for each File Gateway
  * Most recently used data is **cached** in File Gateway
  * Can be **mounted** on many servers
* **Volume Gateway** - 
  * Block storage using **iSCSI protocol** backed by S3
  * **EBS snapshots** stored **S3** buckets
  * **Cached Volume** - Recently used data cached
  * **Stored Volume** - Entire dataset in premise with scheduled backups in S3
* **Tape Gateway** - 
  * Backup from on premise **tape to S3 Glacier** using **iSCSI protocol**

## Athena

* **Serverless** service
* Allows to do analytics directly on **S3**
* Supports **data formats** CSV, JSON, ORC, Avro, Parquet (built on Presto)
* Uses **SQL** to query the files
* **Good for** analyzing VPC Flow Logs, ELB Logs etc.

## IAM

* **Global** service
* **Users** for individuals
* **User Groups** for grouping users with similar permission requirements
* **Roles** are for machines or internal AWS resources. One IAM Role for ONE application

## EC2

* **Security Groups** are for network security
* **Security group**s are locked down to a region / VPC combination
* **Security Groups** can refer to other Security Groups
* One **security group** can be attached to multiple EC2 instances
* Multiple **security groups** can be assigned to a single EC2 instance
* If connection to application times out, it could be a **security group** issue. If connection is refused, it's an application issue
* **Security Group** - All inbound traffic is blocked by default
* **Security Group** - All outbound traffic is authorized by default
* **Security Groups** are stateful - if the inbound traffic on a port is allowed, outbound traffic on the same port is automatically allowed
* Change in **security group** takes effect immediately
* **Security groups** cannot blacklist an IP or port. Everything is blocked by default, we need to specifically open ports
* **Elastic IP** gives a fixed IP to an EC2 instance across restarts
* By default one AWS account can have 5 **elastic IP**
* Prefer load balancer over **Elastic IP**
* EC2 **User Data** script runs once (with root privileges) at the instance first start
* EC2 Launch types - 
  * **On-demand instances** - short workload, predictable pricing
  * **Reserved instances** - long workload (>= 1 year)
  * **Convertible reserved instances** - long workload with flexible instance types
  * **Scheduled reserved instances** - reserved for specific time window
  * **Spot instances** - short workload, cheap, can lose instances, good for batch jobs, big data analytics etc.
  * **Dedicated instances** - no other customer will share the hardware, but instances from same AWS account can share hardware, no control on instance placement
  * **Dedicated hosts** - the entire server is reserved, provides more control on instance placement, more visibility into sockets and cores, good for "bring your own licenses", complicated regulatory needs
* **Billing** by second with a minimum of 60 seconds
* A custom **AMI** can be created with pre-installed software packages, security patches etc. instead of writing user data scripts, so that the boot time is less during autoscaling
* **AMIs** are built for a specific region, but can be copied across regions
* T2/T3 are **burstable** instances. Spikes are handled using burst credits that are accumulated over time. If burst credits are all consumed, performance will suffer
* **M instance types** are balanced
* `http://169.254.169.254/latest/user-data/` gives **user data** scripts
* `http://169.254.169.254/latest/meta-data/` gives **meta data**
* `http://169.254.169.254/latest/meta-data/public-ipv4/` gives **public IP**
* `http://169.254.169.254/latest/meta-data/local-ipv4/` gives **local IP**
* Two types of **placement groups**
  * **Clustered Placement Group** - Grouping of instances within a single AZ. Recommended for applications that need low network latency and high network throughput. Only certain instances can be launched in this placement group. It cannot spac multiple AZ
  * **Spread Placement Group** - Instances are placed in distinct hardware. Recommended for applications that have a small number of critical instances that should be kept separate from each other. It can span multiple AZ
* The instances within a **placement group** should be homogeneous
* Existing instances can't be moved into a **placement group**
* **Placement groups** can't be merged

## EFS

* Supports Network File System version 4 (**NFSv4**)
* **Read after write** consistency
* Data is stored across **multiple AZ's** within a region
* **No pre-provisioning** required
* To use EFS
  * install **amazon-efs-utils**
  * **mount** the EFS at the appropriate location

## ELB
  
* **ELB types** - 
  * **Classic Load Balancer** (V1 - old generation) - Lower cost than ALB, but less flexibility
  * **Application Load Balancer** (V2 - new generation) - Layer 7 - application aware
  * **Network Load Balancer** (V2 - new generation) - Layer 4 - extreme performance
* ELB provides **health check** for instances
* **ALB** can handle **multiple applications** where each application has a traget group and load for that application is balanced across instances within the particular target group
* ALB supports **HTTP/HTTPS** & **Websocket** protocols
* ALB - True IP, port and protocol details of the client are inserted in HTTP headers - **X-Forwarded-For**, **X-Forwarded-Port** and **X-Forwarded-Proto** respectively
* ALB can **route** based on hostname in URL and route in URL
* **Network Load Balancers** are mostly used for extreme performance and should not be the default load balancer
* **Network Load Balancers** have less latency ~100 ms (vs 400 ms for ALB)
* Load Balancers have **static host name**. DO NOT resolve & use underlying IP
* LBs can scale but not instantaneously – contact AWS for a “warm-up”
* ELBs do not have a predefined IPv4 address. We resolve to them using a **DNS name**
* **504 error** means the gateway has timed out and it is an application issue and NOT a load balancer issue
* **Sticky session** - required if the ec2 instance is writing a file to the local disk. Traffic will not go to other ec2 instances for the session
* **Cross zone load balancing** - If one AZ does not receive any traffic
* **Path Patterns** - Allows to route traffic based on the URL patterns

## Auto Scaling Group

* Auto scaling group is configured to register new instances to a traget group of **ELB**
* **IAM role** attached to the ASG will get assigned to the instances
* If instance gets **terminated**, ASG will restart it
* If instance is marked as **unhealthy** by load balancer, ASG will restart it
* ASG can scale based on **CloudWatch alarms**
* ASG can scale based on **custom metric** sent by applications to CloudWatch
* If all subnets in different **availability zones** are selected, the ASG will distribute the instances across multiple AZ
* During the configured warm up period the EC2 instance will not contribute to the **auto scaling metrics**
* **Scaling out** is increasing the number of instances and **scaling up** is increasing the resources

## EBS

* An EBS volume is a **network drive**
* An EC2 machine by default loses its **root volume** when terminated
* It's locked to an **AZ**. To move a volume to a different AZ, a snapshot needs to be created
* EBS Volume types:
  * **General Purposse SSD (GP2)** - General purpose SSD volume
  * **Provisioned IOPS SSD (IO1)** - Highest-performance SSD volume for mission-critical low-latency or high throughput workloads. Good for Databases.
  * **Throughput Optimized HDD (ST1)** - Low cost HDD volume designed for **frequently accessed**, throughput intensive workloads. Good for big data and datawarehouses
  * **Cold HDD (SC1)** - Lowest cost HDD volume designed for **less frequently** accessed workloads. Good for file servers
  * **EBS Magnetic HDD (Standard)** - Previous generation HDD. For workloads where data is infrequently accessed
* **RAID 0**
  * Striping in multiple disk volumes
  * When I/O performance is more important than fault tolerance
  * Loss of a single volume results in complete data loss
* **RAID 1**
  * When fault tolerance is more important than I/O performance
  * Even in the absence of RAID 1, EBS is already replicated within AZ
* **Max IOPS/Volume**
  * io1 - 64,000 (based on 16K I/O size)
  * gp2 - 16,000 (based on 16K I/O size)
  * st1 - 500 (based on 1 MB I/O size)
  * sc1 - 250 (based on 1 MB I/O size)
* SSD is good for short random access. HDD is good for heavy sequential access
* SSD provides high IOPS (np. of read-write per second). HDD provides high throughput (no. of bits read/written per second)
* The size and IOPS (**only for IO1**) can be increased
* Increasing the size or the volume does not automatically increase the **size of the partition**
* EBS volumes can be backed up using **snapshots**
* **Snapshots** are also used to resizing a volume down, changing the volume type and encrypting a volume
* **Snapshots** occupy only the size of data
* **Snapshots** exist on S3
* **Snapshots** are incremental
* **Snapshots** can be shared, but only if they are unencrypted
* **Snapshots** of encrypted volumes are always encrypted
* Volumes restored from encrypted **snapshots** are encrypted automatically
* To take a **snapshot** of the root device, the instance needs to be stopped
* Copying an unencrypted **snapshot** allows encryption
* EBS **Encryption** leverages keys from KMS (AES-256)
* All the data in flight moving between the instance and an encrypted volume is **encrypted**
* Encryption of a root volume involves following steps, 
  * Take a **snapshot**
  * Copy the **snapshot** and choose the ecryption option (Once encrypted, it cannot be uncrypted by again making a copy)
  * Create an Image (**AMI**) from the encrypted snapshot
* EBS **backups** use IO and hence backups should be taken during off-peak hours
* Each EBS volume is automatically **replicated** within its own AZ to protect from component failure and provide high availability and durability
* EC2 instance and its volume are going to be in the same **AZ**
* Migrating EBS to a different **AZ or Region** involves the following steps
  * Create a **snapshot**
  * Create an **AMI**
  * Copy the **AMI** to a different Region
  * Launch an **EC2 instance** in a different Region with the AMI
* The **size and type** of the EBS volumes can be changed without even stopping the EC2 instance
* **AMI** can be created directly from the volume as well
* **AMI** root device storage can be
  * Instance Store (Ephemeral Stores)
  * EBS Backed Volumes
* **Instance store** volumes are created from a template stored in Amazon S3
* **Instance stores** are attached to the host where the EC2 is running, whereas EBS volumes are network volumes. However, in 90% of the use cases the difference in latency with the two types of stores does not make any difference
* Throughput = IOPS * I/O size. The I/O size is 256KB (earlier 16KB). If the IOPS provisioned is 500, the instance can achieve 500 * 256KB writes per second
* EBS Optimized Instances - With small additional fee, customers can launch certain Amazon EC2 instance types as EBS-optimized instances. EBS-optimized instances enable EC2 instances to fully use the IOPS provisioned on an EBS volume. Contention between Amazon EBS I/O and other traffic from the EC2 instance is minimized

## CloudWatch

* CloudWatch is for monitoring performance, whereas **CloudTrail** is for auditing API calls
* CloudWatch with EC2 will monitor events every 5 min by default. With **detailed monitoring**, the interval will be 1 min
* CloudWatch alarms can be created to trigger notifications
* Enabling CloudWatch logs for **EC2**
  * Assign appropriate CloudWatch access policy to the IAM role
  * Install CloudWatch agent (awslogsd) in EC2
* Since AWS does not have access to the  underlying OS, some metrics are **missing** including disk and memory utilization
* CloudWatch can collect metrics and logs from services, resources and applications on AWS as well on-premise services
* CloudWatch Alarms can be created to send notifications or do autoscaling when a certain metrics satisfies a configured condition
* CloudWatch Events allow the user to configure a Lambda function to be triggered on certain system events viz. launch of EC2 instance in an ASG

## Route 53
  
* In AWS, the most common records are:
  * **A**: URL to IPv4
  * **AAAA**: URL to IPv6
  * **CNAME**: URL to URL
  * **Alias**: URL to AWS resource.
* Prefer **Alias** over CNAME for AWS resources (for performance reasons)
* Route53 has advanced features such as:
  * **Load balancing** (through DNS – also called client load balancing)
  * **Health checks** (although limited…)
  * **Routing policy**: simple, failover, geolocation, geoproximity, latency, weighted, multivalue answer
* IPv4 - 32 bit, IPv6 - 128 bit
* **Simple Routing** - Multiple IP addresses against a single A record. Route 53 returs all of them in random order
* **Wighted Routing** - A separate A record for each IP with a percentage weight. A separate health check can be associated with each IP or A record. SNS notification can be sent if a health check fails. If a health check fails, the server is removed from Route 53, until the health check passes
* **Latency Based Routing** - A separate A record for each IP with a percentage weight. A separate health check can be associated with each IP or A record. Routing happens to the server with lowest latency
* **Failover Routing** - 2 separate A records - one for primary and one for secondary. Health check can be associated with each, If primary goes down, traffic will all be ruted to secondary
* **Geolocation Based Routing** - A separate A record for each IP. Each A record is mapped to a location and the routing happens to a specific server depending on which location the DNS query originated. Good for scenarios where different website will have different language laels based on location
* **Multivalue Answer** - Simple routing with health checks of each IP
* **Geoproximity** - Must use Route 53 Traffic Flow. Routes traffic based on geographic location of users and resources. This can be further influenced with biases

## RDS

* Upto 5 **Read Replicas** (Async Replication - within AZ, cross AZ or cross Region)
* **Read replicas** of read replicas are possible
* Each **read replica** will have its own DNS endpoint
* **Read replica** can be created in a separate region as well
* If a **read replica** is promoted to its own database, the replication will stop
* **Read replica** cannot be enabled unless the automatic backups are also enabled
* **Read replicas** themselves can be **Multi-AZ*** for disaster recovery
* A failover in a **Multi-AZ** deployment can be forced by rebooting the DB
* Two ways of improving performance
  * Read replicas
  * ElasticCache
* **Replication** for Disaster Recovery is synchronous (across AZ - Automatic failover - DNS endpoint remains same) - Multi AZ
* **Replicas** can be promoted to their own DB
* Automated **backups**:
  * Daily full snapshot of the database
  * Capture transaction logs in real time
  * Ability to restore to any point in time
  * 7 days retention (can be increased to 35 days)
* The **backup** data is stored in S3
* Backups are taken during specified window. The application may experience elavated latency during **backup**
* Restoring DB from automatic **backup** or snapshots always creates a new RDS instance with a new DNS endpoint
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
* RDS, in general, is **not serverless** (except Aurora Serverless which is serverless)
* We cannot access the RDS **virtual machines**. Patching the RDS operating system is Amazon's responsibility

## DynamoDB

* Supports both **document and key-value** data model
* Stored on **SSD storage**
* **Spread across** 3 geographically distributed data centers
* Supports both **Eventual Consistant** Reads (Default) & **Strongly Consistant** Reads
* **Serverless** service

## Redshift

* Amazon's **data warehouse** solution
* Single node (160 GB) or multi node (leader node and compute node - upto 128 compute nodes)
* **Column based** data store, column based compression techniques and multiple other compression techniques
* No indexes or materialized views
* Massively **parallel processing**
* Redshift attempts to maintain **3 copies** of data (the original and replica on the compute nodes and a backup in S3)
* Available in only **1 AZ**
* **Backup** retention period is 1 day by default which can be extended to 35 days
* Can asynchronously replicate to S3 in a different region for **disaster recovery**

## Aurora

* Aurora **storage** automatically grows in increments of 10GB, up to 64 TB
* Aurora can have 15 **replicas** while MySQL has 5, and the replication process is faster (sub 10 ms replica lag)
* 2 copies of data is maintained in each **AZ** with a minimum of 3 AZ
* Compute resources can scale upto 32 vCPUs and 244 GB of memory
* Aurora can transparently handle the loss of 2 copies of data without affecting write availability and 3 copies of data without affecting read availability
* **Backups** and snapshots does not impact database performance
* Storage is self-healing. Disks and blocks are scanned for errors and repaired automatically
* Aurora snapshots can be shared with other AWS accounts
* Two types of **replicas** - MySQL replicas (based on MySQL binlog) and Aurora Replicas
* Automated **failover** is only possible with Aurora replicas (not MySQL replicas)
* **Failover** in Aurora is instantaneous. It’s HA native

## ElasticCache

* ElastiCache is to get managed Redis or Memcached
* ElasticCache features - 
  * Write Scaling using **sharding**
  * Read Scaling using **Read Replicas**
  * **Multi AZ** with Failover Capability
* Redis - Multi AZ, **Backups and restore**
* Memcached - Multi threaded, horizontal scaling
* Elastic cache can be used as - **db cache**, **session store**
* Caching patterns - 
  * Write through
  * Lazy loading
  
## VPC

* VPC **Flow Logs** allows us to monitor the traffic within, in and out of your VPC (useful for security, performance, audit)
* VPC are per Account per **Region**
* Subnets are per VPC per **AZ**
* Subnet doesn't span across **AZ**
* **Security Groups** doesn't span across VPC
* A VPC can have only 1 **internet Gateway**
* Amazon reserves 5 **IP** in each subnet
* Each EC2 instance performs source/destination checks by default. This means that the instance must be the source or destination of any traffic it sends or receives. However, a **NAT** instance must be able to send and receive traffic when the source or destination is not itself. Therefore, we must disable source/destination checks on the NAT instance
* **NAT** instace / gateway must be in public subnet
* A route from private subnet to **NAT** Gateway is important
* **NAT** instance must have a security group
* **NAT** Gateway is redundant inside an AZ
* **NAT** Gateway starts at 5 Gbps and scales upto 45 Gbps
* **NAT** Gateway don't need a security group
* **NAT** Gateway automatically have a public IP assigned
* With **NAT** Gateway there is no need to disable source / destination checks
* Create a **NAT** Gateway in each AZ and configure the route to use the NAT Gateway in the same AZ
* The public subnet must be configured to asign public **IP** addresses to the EC2 machines
* **NACL** is evaluated before security groups
* **NACL's** rules are executed in chronological order with lowest numbered rule evaluated first. Therefore DENY should come before ALLOW
* Default **NACL** allows all inbound and outbound traffic
* Custom **NACL** by default denies all inbound and outbound traffic
* A subnet can be associated with only one **NACL** and one NACL can be assigned to multiple subnets
* **NACLs** are stateless unlike security groups
* Default **NACL** allows all outbound and inbound traffic
* By default, each custom **NACL** denies all inbound and outbound traffic
* Each subnet in a VPC must be associated with a **NACL**. If we do not assign an NACL with the subnet, the subnet will have the default NACL assigned
* IP addresses can be blocked by **NACL** and NOT security groups
* 2 public subnets are required to create a **load balancer**
* VPC **flow logs** capture information about IP traffic going to and from the network interfaces in the VPC
* **Flow logs** are stored in Cloudwatch logs
* **Flow logs** can be created at VPC level, subnet level or network interface level
* **Flow logs** cannot be enabled for VPCs that are peered with our VPC unless the peered VPCs belong to our AWS account
* **Flow logs** cannot be tagged
* Once a **Flow log** is created, its configuration cannot be changed
* Not all traffic is monitored in VPC **Flow log**. Traffic not monitored include:
  * DHCP traffice
  * Traffic to and from Amazon DNS
  * Traffic of Amzon Windows License activation
  * Traffic to and from 169.254.169.254 for instance metadata
  * Traffic to the reserved IP address for the default VPC router
* A **bastion host** is a special purpose computer specially designed to withstand attacks. The computer usually hosts a single application, e.g. a proxy server, and all other services are removed or limited to reduce the threat to the computer. It is hardened in this manner primarily due to its location and purpose which is either on the outside of a firewall or in a demelitarized zone (DMZ) and usually involves access from untrusted networks or computers
* A **NAT** Gateway is used to provide internet traffic to the private subnet
* A **Bastion host** is used to administer the EC2 instances in the private subnet using SSH or RDP 
* Direct Connect provides reliable, high throughput, dedicated and secure connection from the local data center to the AWS
* **VPC endpoints** allow the VPC to privately connect to the supported AWS services without leaving the AWS network. Two types of VPC Endpoints - 
  * **Interface Endpoints** - An elastic network interface with a private IP address that serves as an entrypoint for traffic destined to a supported service
  * **Gateway Endpoints** - Only for S3 and DynamoDB
* On creation of a VPC, a default route table, NACL and security group are automatically created. Subnets and Internet Gateways are not automatically created
* US-East-1A in one AWS account can be completely different from US-East-1A in another AWS account
* **Traffic Flow** - 
  * Internet Gateway -> Router -> Route Table -> NACL -> Security Group -> NAT Instance -> EC2 in private subnet
  * Internet Gateway -> Router -> Route Table -> NACL -> NAT Gateway -> EC2 in private subnet
* To be able to SSH into an EC2 system in a public subnet of a custom VPC, following are required
  * An **internet gateway** should be assigned to the VPC
  * The **public subnets** should be associated with a custom route table that should have a route that will allow destination to everywhere (0.0.0.0/0) through the internet gateway
* **Private subnets** should be associated with a custom NACL that allows traffic to and from the public subnets (atleast SSH & ICMP) and internet (for NAT Gateway to work)
* **Private subnet** should be associated with a route table that route all internet traffic (0.0.0.0/0) to the NAT Gateway
* VPC **peering** connection does not support edge to edge routing
* A Corporare network can be connected to a VPC using a VPN over the internet or a VPN over AWS Direct Connect
* Using AWS Direct Connect, a dedicated network connection between the AWS VPC and corporate network can be established

## SQS

* SQS is **pull based**, NOT push based
* Messages are 256 KB in size
* Messages can be kept in the queeu from 1 minute to 14 days; the default retention period is 4 days
* Visibility Timeout is the amount of time that the message is invisible in the SQS queue after a reader picks up that message. If the message is processed successfully before the timeout expires, the message will be deleted from the queue. Otherwise, the message will again become visible after the timeout for another reader to pick it up for processing. This could result is message being delivered twice
* Visibility Timeout maximum is 12 hours
* Standard SQS guarantees that the message will be delivered at least once
* SQS Long Polling API call doesn't return until a message arrives in the queue or the long poll times out. This result in less number of API calls and thus less cost
* Standard queue lets us have a nearly unlimited number of transactions per second
* Standard queue may deliver more than one copy of the same message
* Standard queue provides best effort ordering; out of order delivery is possible
* FIFO queue strictly preserves order
* FIFO queue gurantees exactly-once delivery
* FIFO queue is limited to 300 transactions per second
* FIFO queues also support message groups that allow multiple ordered message groups within a single queue
* It's a way to decouple infrastructure


## SWF

* Workflow service viz. human interaction
* SWF workflow execution can last upto 1 year
* SWF provides a task oriented API, whereas SQS peovides a message oriented API
* Workflow Startes, Deciders, Activity Workers

## SNS

* Push notifications to mobile devices
* SMS, email & HTTP endpoints
* Push based delivery, no polling
* Stored redundantly across multiple AZ
* Supports multiple topics

## API Gateway

* Supports caching of API response
* Allows enabling CORS to access multiple AWS resources with different origin name using Javascript
* Allows logging result to CloudWatch
* Allow throttling to prevent attacks
* CORS is enforce by client's browser

## Kinesis

* Amazon alternative of Kafka
* Streaming data
* Types of Kinesis
  * Kinesis Streams
  * Kinesis Analytics
  * Kinesis Firehose
* Default retension period 24 hours
* Maximum retention period 7 days
* Kinesis Shard Read - 5 transactions per second upto a total data  rate of 2MB per second
* Kinesis Shard Write - 1000 records per second upto a maximum data write of 1 MB per second
* The total capacity of the stream is the sum of the capacities of its shards
* Kinesis Streams have shards
* Kinesis Firehose doesn't have a persistent store. As soon as the data comes in, the data needs to be processed optionally using Lambda functions and send it to the appropriate data stores

## Cognito

* User pool consists of user data like email, userid etc. It handles authentication, registration, recovery etc.
* Identity pools are temporary IAM roles to access various AWS resources
* Cognito uses push synchronizations and SNS notifications to push updates across devices
* Cognito is an Identity broker which handles interaction between the AWS applications and the Web Id Provider

## AWS OpsWorks

* Managed **configuration management** system
* Provides managed instancess of **Chef** and **Puppet**

## CodeDeploy

* **EC2/on-premise Deployment Configuration**
  * All At Once
    * In-place deployments - Deployment in all the EC2 instances will be done at the same time
    * Blue-Green deployments - A replacement environment will be created and the triffic will be moved from the old to the new environment all at once
  * Half At A Time
    * In-place deployments - self explanatory
    * Blue-Green deployments - self explanatory
  * One At A Time
    * In-place deployments - self explanatory
    * Blue-Green deployments - self explanatory

* **Lambda Deployment Configuration**
  * **Canary** - Traffic is shifted to the new Lambda version in two increments. The percentage of traffic and the time interval between the increments are configurable
  * **Linear** - Traffic is shifted to the new Lambda version in equal increments with equal number of minutes between each increment. The percentage of traffic in each increment and the time interval between each increment are configurable
  * **All at once** - Traffic is shifted to the new Lambda version all at once


