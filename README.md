# AWS SAA preparation notes

## General
 - Regions -> Geographical locations with multiple DCs
 - Availability Zone (AZ) -> DC per region, most of regions have more then 1 DC (city wise)
 - Find in: https://infrastructure.aws 
 - Choose regions based on complaince, latency, service availability per region or pricing (different prices)
 - AZ -> mostly 3 per regions (2-6 range)
 - AZ have redundancy in power, DC, building, dissaster recovery
 - AZ are interconnected by ultra high speed link and low latency

 - AWS Edge - is a CDN (Cache) in more cities located, to lower latency
 - Some services are region scoped and some are global (IAM)

 - Options to communicate with AWS
  - GUI - AWS Management Console
  - AWS CLI - for CLI - protected by Access Key (Token based auth)
  - AWS SDK - for coding 

## IAM - Identity and Access Management
 - Global service
 - All user accounts are created here (including root)
 - Users should be created for every user and grouped to specific groups
 - Groups only contain users
 - User can exist without group
 - User can be part of multiple groups

 - After we assign permissions
 - Permission with policies
 - Policies are JSON based docs, describing what services and what there they can do
 - Apply least privilege policy (minimal access as possible) 

 - You can change account alias (right top in IAM User menu) - this can be used during login
 - user format after login is: <username>@<account alias>
 - Policy format:

```
{
 "Version": <data>,
 "Id": <ID>,
 "Statement": [
 {
    "Sid": <sub ID>,
    "Effect": <allow/deny>,
    "Principal" : {
      "AWS": ["arm:aws:iam::123434:root"]
      },
    "Action": [
      "s3:GetObject",
      "s3:PutObject"
    ],
    "Resource": ["arn:aws:s3:::mybucket/*"]
    }
  ]
}
```

 - Strong Password can be forced by Password Policy
 - Password policy also add option to change passowrd after time, if user can change it or not
 - MFA - MultiFactor Auth - protect users with MFA device
 - Virtual MFA device - Google Authenticator, Authy, Duo, ...
 - HW MFA device - USB security key - e.g. YubiKey (by Yubico) 
 - HW Key FOB MFA Device - like pager
 - Or goverment level MFA device

 - AWS CLI example:
```
 # aws configure
 AWS Access Key ID [None]: <enter here>
 AWS Secret Access Key [None]: <enter here>
 Default region name [None]: <region>
 Default output format [None]: <skipable>

 # aws iam list-users
 <ommited>

 # aws s3 cp file.txt s3://mybucket/file.txt
 <ommited>
```

### IAM Security tools
 - IAM Credentials Report 
   - reports list of accounts and status of credentials
   - its a download excel type of thing
   - password last used, changed, number of access keys, password enabled

 - IAM Access Advisor
   - Checking least priviledge permsission and advise to remove some if users are not using them
   - showing which service was access when, and if something was not used, you can remove the permission

### AWS Cloud shell
 - Terminal via GUI to apply aws CLI in browser mode
 - Top right icon in main screen of console with terminal icon
 - Not every region supports that
 

### IAM Roles
 - Temprorarly permisions assigne to some resources (like EC2)
 - In case that EC2 needs to perform some operation in AWS (like needs to have role to create more EC2)
 - Commonly used with EC2 roles, lambda roles, CloudFormation
 - Roles can be attached/detached on the fly to allow temporarly enhance the permissions



## EC2 - Elastic Cloud Computing 
 - Infrastructure as a Service
 - Virtual Machines
 - We choose:
   - OS - linux, windows...
   - CPUs, RAM
   - Storage
   - Networking
   - Security Groups (FW)
   - Bootstrap script (user data)

 - Bootstraping is done by EC2 User data
 - We use for install software, download thing during boot
 - More User Date === longer boot time
 - It is very common to terminate instances
 - After Stop -> Start , public IPv4 is changing
 - Stopped instances are not charged

### EC2 Instance Types
 - m5.3xlarge - m=instance class, 5=generation, 2xlarge=size(CPU,memory)

 - Different types:
   - General Purpose - Compute, Memory and networking balance
   - Compute Optimized - CPU optimized, Used for: Media coding, High Performance Computing, Batch processing, Machine Learning, Gaming server, ...
   - Memory Optimized - Fast memory workload, Used for: BigData, InMemory DB, Distributed Cache, Application performin in memory
   - Storage Optimized - Storage intensive tasks, high IO, Used for: Relational DB, cache,....

### Security Group
  - Basic networking security in AWS
  - Traffic flow control
  - only allow is used
  - Reference for source can be IP or security group
  - Firewall
  - We can control inbound and outboud traffic 
  - Can be attached to multiple instnces
  - VPC and Region scoped
  - Its good to have security groups for some add hoc services, like ssh
  - Timeout mostly represents security block issue
  - By default - ingress  traffic is blocked - outbound traffic is all allowed

### EC2 Purchasing options
  - On Demand - classic use when needed - most expensive, paying for uptime, no upfront payment
  - Reserved: (1-3 years) 75% discount - higher discount for longer time and paying upfront
     - Reserved Instances - long workloads, like custom DB  - same type is reserver for entire time
     - Convertible Reserved Instances - long workload + flexible - Up to 54% discount, we can change type from e.g.t4.micro -> t4.small 
     - Schedulled Reserved - run on a specific time per day, or daily 1h 
  - Spot Instances - cheap but not stable, we may loose them - up to 90% discount, spot price is changing over time and we define our max, if spot price is higher then max, we can losse
     - Good for batch and ongoing processing where loosing data is not a problem
  - Dedicate hosts - buy a physical server for specific time (special licenses and so) or used for complaince requirements
     - expensive
  - Dedicated instance - HW below is sharing instances only for same account

### EC2 Spot Instances
  - Get 90% discount
  - Define max spot prices (our max what we are willing to pay)
  - If current spot price < max price = we have that instance, it depends on offers and capacity
  - If current spot price > max price = we can choose to stop or terminate with 2min grace period
  - Persistent means that if instance is gone, it will try to create another
  - Delete means -> 1. Delete spot request to not recreate instances -> 2. delete isntances (terminate)

  - `Spot Block`
  - EOL - not supported, good to know for exam
  - Block spot instance for specific timeframe 1-6h without interuptions

  - `Spot Fleets`
  - Combination of Spot Instances + Ondemand Instances
  - Allow to request Spot instances from pools for best price
  - Multiple instances types = launch pools
  - Spot fleets will get the best prices instance type 
  - Strategies:
    - lowestPrice = Lowest cost
    - diversified = distributed accorss all pools - better availability and used for longer wokrloads
    - capacityOptimized - from pool get the best capacity instances for max savings

### Placement Groups
  - Strategy to plant instances 
  - Stratiegies:
    - `Cluster` - Single AZ and same Rack - Low latency, low HA
    - `Spread` - Critical Apps, Spread accross HW, max 7 per AZ, can spread to multpiple AZ, maximize HA
    - `Partition` - spread instances accross different partitions (racks) within AZ, up to 100 instancces per group, partitions can be part of same or diff AZ

### Elastic Network Interfaces ENI
  - Represent a NIC card (vNIC)
  - It can be detached and attached and ensure to keep the IP address, security groups or macs
  - AZ limited
  - Its created by default for instances, but its not permanent
  - Fr permanent, create separately

### EC2 Hibernate
  - Save the memory state to disk and stop the instance
  - When you start, it resume and you continue
  - Its written to root EBS volume 
  - the root EBS must be encrypted
  - can be kept for max 60 days (can change)

### EC2 Nitro
  - Next gen EC2
  - New virtualization technology
  - Faster networking
  - Higher EBS speed IOPS, non Nitro 32000 / Nitro 64000
  - Better securty
  - Latest instances have Nitro

### EC2 vCPU
  - 1x vCPU = 1 Thread on 1 Core
  - Reservation for full core is possible
  - Tune number of cores and threads during creation 

### EC2 capacity reservation
  - Reserve for custom perios (non 1year)
  - Number of AZ

## Elastic Block Device EBS
  - Network drive attached to EC2
  - Allow to keep data even after termination
  - Single attach, in some special types, Multi Attach is supported
  - Limited to AZ
  - When creating volumes, ensure that it is in same AZ

#### EBS Snapshot 
  - Save staete and data from point in time
  - Used for backups
  - Volume is not needed to be detached
  - Copy snapshots between AZ
  - We can create new volumes from EBS -> this is how we can move volumes between AZ

### Amazon Machine Image
  - Same as qcow2 
  - special customized image for Ec2
  - Region scoped
  - There publicly available or create custom or by in Marketplace
  - You can create it from EC2
  - Right Click -> Images and Templates -> Create image
  - Creating AMI makes faster booting then using User Data

### EC2 Instance Store
  - Higher performance the Network Drives
  - Better I/O
  - It is default instance storage
  - Not feasible for long term storage

### EBS Volume Types
  - GP2/GP3 - SSD - General Purpose - balance price and performance - 1-16TB 
    - GP2 - IOPS up to 3000 - max 16000 (based on size) - 3IOPs per GB
    - GP3 - Base IOPS 3000 - Up to 16000 - better throughput 1000 MiB/s - independently set IOPS (not in GP2)
  - IO1/IO2 - SSD - Highest Performance - mission critical or high-throughput workloads
    - Critical business apps
    - 4-16TiB
    - Default max IOPS 32000
    - Max IOPS for EC2 Nitro = 64000
    - Define IOPS independently from storage size
    - io2 better latency (sub milisecond)
  - st1 - HDD - Low cost HDD - frequently access, low throughput
    - Big Data
  - sc1 - HDD - Lowest cost - less frequently access 

### EBS Multi-Attach
  - Same volume attached to multiple EC2
  - Full read/write to disk
  - IO1/IO2 can be used only
  - Application must manage concurent writes 
  - Filesystem must support clustering (not xfs)

### EBS Encryption
  - Must be enabled explicitly
  - Data at rest is encrupt
  - All data moving over network is encrypted
  - All snapshots encrypted
  - All volumes created from snapshot encrypted
  - Minimal impact on latency
  - Using AES-256 

  - Encrypt unencrypted EBS volume
  - Create uncencrypted snapshot -> copy the snapshot and encrypt -> create new volume encrypted

## Elastic File System - EFS
  - Amazon create NFSv4.1 - Not in exam
  - Managed NFS (managed by AWS)
  - Mountable to multiple to EC2
  - 3x more expensive as GP2
  - Highly available and scalable
  - pay for usage only
  - encryptable by KMS
  - mountable by 100 EC2 instances accross AZ
  - Good for web sharring 

## High Availability & Scalability
  - Vertical scalability - change flavor
  - Horizontal scalability - add more EC2 instances

## Elastic Load Balancer ELB
  - Load balancer in front of EC2 instances
  - Sprad load accross EC2 instances
  - Handles failures of EC2 - by probing
  - Do healthchecks regularly
  - SSL termination
  - HA accross zones
  - AWS Managed (AWS upgrades, manage, take care)
  - Integrated with EC3, ACM, Cloudwatch, S3

  - Health check can be L3 or L4 or L7
  - Send url request to port and wait for status code 200
  - Security groups can be used in security groups to identify and flexibly attach EC2 to ELB

  - OutOfService for instance can be problem: Security group or Health check URL is not giving 200

### Classic Load Balancer CLB
  - 2009 - Deprecated
  - Supports HTTP,HTTPS TCP, SSL 
  - Can have instances from various AZ and do load balancing between AZ

### Application Load Balancer ALB
  - 2016
  - Supports HTTP, HTTS, Websocket
  - Load balancing between target groups
  - Support load balancing between containers
  - Load balancing based on URL or Hostanme or Query Strings in URL
  - `Target Groups` === EC2 instances, ECS tasks, Lambda Functions, IP addresses
  - It has a fixed hosntame
  - Client IP (endd user) is not visible to EC2 directly, but its there is in header (X-Forwarder-For, X-Forwarder-Port & X-Forwarder-Proto)
  
### Network Load Balancer NLB
  - 2017
  - Support TCP, TLS, UDP
  - High performance - milions of requests per second
  - Low latency ~ 100ms
  - Supports to have one static IP per AZ - can be Elastic
  - Using target groups as well: EC2, IP address, Application Load balancer (chaining)

### Gateway Load Balancer GLB
  - 2020
  - Operates at L3, IP
  - Load balancing in 1 arm mode
  - Traffic is forwarded to IDS, IPS, 3rd party FWs, Caches, ...
  - Combines: Transparent Network GW and Load Balancer
  - Using GENEVE Protocol on port 6081

### Sticky Sessions
  - Multiple requests from same client will go to same instance
  - ALB and CLB default behaviour
  - Cookie used for stickiness has an expiration date
  - 2 kind of cookies:
    - Application Based 
      - custom cookie - cookie name specified per target group
      - application cookie = AWSALBAPP, generated by AWS ALB
    - Duration based
      - Same as application based, just custom duration specified

### Cross Zone Load Balancing
  - Every instance is having same amount of load
  - Between 2 AZ, 2 ALB can split the load to another AZ instances evenly 
  - ALB enabled by default - free
  - NLB - disabled - paid service if enabled
  - CLB - disbaled - free
  - `Without Cross Zone`:
  - traffic load is distributed evenly between AZ ALB but it does not care about how many instances the AZ have
  - regardless of instance number !!

### SSL Termination
  - SSL certificates are loaded on LB
  - ACM can be used to manage certificates
  - SNI = Server Name Indication
  - SNI allow to add more certificates and use per URL
  - Client must use hostname when initiate SSL protocol
  - Only ALB or NLB supported

### Connection Draining / Deregistration delay
  - Draining = CLB
  - Delay = ALB/NLB
  - If instance is unhealthy, we give time to complete requests before disconnect
  - value betwee 1 - 3600 sec 
  - can be disabled, default 300s
  - set based on how long it takes to complete some requests

## Autoscaling Groups ASG
  - Scale out and scale in of instances
  - Can specify max and min number of instances
  - Automation of instance creation based on need or load
  - Minimum = will alway run
  - Max = how much is max for instances
  - Load balancer will monitor the AutoScaling group instances
  - ASG = Attributes are the same or close as running an EC2 + other info like min max, scaling, load balancer ..
  - Scale based on alarms triggered by CloudWatch
  - Alarm can be custom metric:
    - Average CPU
    - Network traffic in/out
    - number of frequests
    - custom metrics - number of connected users
  - Can use launch configuration or launch templates
  - IAM role most be attached to ASG to create instances
  - Unhealthy VMs marked by LB can be automatically terminated
  - Scaling Policies:
    - 1. Target Tracking Scaling - Easy - check average value, like CPU around 40%, create/destroy VMs to keep that value
    - 2. Simple Scaling - define max and min - like max cpu 70% -> create instance / less then 30% cpu -> terminate
    - 3. Scheduled Actions - scaling based on patterns - like during peak hours create instance, or like Black Friday
    - 4. Predictive Scaling - AI driven - based on historical data
  - Metrics to scale on:
    - CPU Utilization
    - Request Count
    - Network In/Out
  - Cool down: period (def. 300s) when no scaling happens to give time to stabilize

### Termination Policy
  - 1. Find AZ with highest instances
  - 2. Find oldest launch config and delete

### Lifecycle hook
  - You can add Pending:Wait state - where you can manual install softw. and after proceed furhter - manualy
  - Same for teminate:pending

## Databases (RDS/Aurora/ElasticCache)

### Relational Database Service RDS
  - Managed SQL query language
  - Similair to mysql
  - Managed by AWS
  - Can create DBs in multiple kind:
    - Postgres
    - Mysql
    - Mariadb
    - Oracle
    - Microsoft SQL
    - Aurora

  - Automated proivsioning
  - Continous backups 
    - automated and enabled by default 
    - full DB backup daily
    - transaction logs are backed up every 5 min
    - Restoration points every 5 min
    - up to 7 days back kept
    - db snapshots = manual backups (retain for a longer time)
  - Monitoring dashboard
  - Read replicas  
    - improves performance
    - Possible to create within AZ, cross AZ, cross Region
    - using ASYNC replication from main RW DB
    - Good for read intensive tasks (reports, analytics) and not impact production DB
    - no payment for cross AZ replication traffic
    - replication fee is there for cross region replication
    - `Can be used from Multi AZ Disaster Recovery`
  - Multi AZ setup
    - Different from Read Replicas is that its SYNC replication and RW
    - Can use the same DNS name so failover of main DB can happen fast
    - It can be just a standby (no active RW traffic can be there, unless failover)
    - From single AZ to multi AZ is no downtime required  - 1 click modify
    - It is created through snapshot and after SYNCing the data
  - Maintanance windows for upgrades
  - Scaling capability
    - Autoscaling - detects and automatically increase the storage if required
    - Supports Max size
    - Scale out if storage is at 90% for 5 min - and 6h hours after last change
    - Good for unpredicatble storage requirements

### RDS Security
  - Encryption at rest by AWS KMS = AES 256
  - Encrypt master and read replicas
  - Master must be encrypted to have Read Replicas encrypted
  - Transparent Data Encryption supported for Oracle and SQL 

  - Encryption at move is doen by SSL
  - Provides SSL options to configure certificates
  - We have to force SSL inside SQL - like Mysql: Grant usage on *.* TO 'user'@'%' REQUIRE SSL;

  - Snapshots are unencrypted if RDS is unencrypted
  - Can be copied and copy can be encrypted
  - This can be used to encrypt DB 
  - Restore from new copy which is encrypted a new DB and delete the old one

  - Recommended to deploy in private segment
  - Leverage security groups (from which EC2 we can access)

  - Access Management is used to limit configuration for specific users
  - IAM based login can be used for RDS Mysql and PosgreSQL
  - EC2 can have IAM role -> used to Get Auth Token -> Use token to login to DB 

## Aurora
  - Closed source - AWS propertiary DB
  - Postgres and Mysql drivers are compatible with Aurora
  - Cloud optimized
  - 5x faster then Mysql on RDS and 3x as postgres
  - It has Autosclaing storage from 10GB -> 128TB
  - It have 15 replicas
  - Failover is instant
  - It costs 20% more then mysql

### Aurora HA
  - 6 copies of data across 3 AZ
    - 4 copies for writes - 3 for reads
  - Automatic replication
  - Data is located on 100s of volumes to avoid single point of failure
  - There is one write endpoint (1 point to contact DB, it can failover)
  - Reader endpoint are there and more of them, used to do queries
  - Automatic failover, backup/recovery, industry compliance , ...

### Aurora Security
  - RDS security is the same as Aurora sec

### Aurora Auto Scaling
  - Multiple replicas - 1 writer , rest are readers
  - If reader will have too many connections or high CPU, it can scale and add more readers

### Aurora Custom Endpoints
  - Create custom endpoints which points to specific DB
  - It will be preffered over the reader 
  - Used for more powerfull reports generation where we target faster instances

### Aurora Serverless
  - Automated Scaling based on actual usage
  - Good if workload is hard to predict
  - No capacity planning needed
  - More costly, payong per second

### Aurora MultiMaster
  - Supports immediate failover between replicas
  - Every node does Read and Write

### Global Aurroa
  - Multi Regoins support
  - We can have multi region replicas - disaster recovery
  - Or Global database 
    - 1 primary ragion RW
    - 5 secondary R0 - lag is 1s
    - max 16 Read replicas per secondary region
    - Failover and promote another region is < 1 min

### Aurora ML
  - Integration to Machine learning
  - Supported DB for various machine learnings
  - Simple,Optimizied and secure integaration to AWS ML services
  - Supported ML:
    - Amazon SageMaker
    - Amazon Comprehend
  

## Amazon ElastiCache
  - Managed as RDS
  - AWS Managed Redis or Memcached
  - in memory DB with high performace and low latency
  - Makes applications stateless
  - No IAM authentication suppoeted
  - IAM Policies are used only for AWS API level security

  - Application first check if data exists in cache
    - if yes, its cache hit - retrieve the data
    - if not, cache miss -  it gets the data from DB and cache will save it 
    
  - Cache should have data invalidate timer
  - Multi User sessions in distributed app require a central cache
  - The app in one EC2 writes data to ElastiCache
  - Another app can read from Cache and reuse it

  - `Redis`
    - Multi AZ with Auto Failover
    - Read Replicas 
    - Data durability
    - Backup and Restore 
    - more like in memory DB
    - Support password/token auth (extra security on top of security groups)
    - Support SSL
  
  - `Memcached`
    - Multinonde paritioning
    - No HA
    - Non persistent
    - No backu and restore
    - Multi-threaded
    - Support SASL based auth
  
  - Cache load possibilites:
    1. Lazy loading - All data is cached, cache data can became stale
    2. Write through - Add or Update data when written to the DB (no stale data)
    3. Session store - store temporay session data with Timer feature

## Route 53 (Amazon DNS service)
  - HA, scalable and AWS Managed DNS service
  - Authorative DNS = CU can update the records
  - Its also a Domain Registrar
  - Supports buying domain in 3rd party and after Amazon will manage it
  - It can do health probes
  - 100% availability in AWS
  - Records:
    - Domain/Subdomain - example.com
    - Record Type - A, AAAA
    - Value - some value 
    - Routing Policy - how to route the responds
    - TTL - amount of time until the record can be cached
  - Host Zones:
    - Public = DNS queries are translated from remote clients (end users)
    - Private = Avaialable inside VPC

  - CNAME - use as alias, only worsk for A or AAAA FQDN
  - Alias - use as alias, works with root domain
          - free of charge
  
### Routing Policies in Route 53
  - Define how to respond to DNS queries
  - `Simple`
    - Route traffic to single resource
    - One DNS record can have 1 or more values
    - Client choose a random one from list
    - AWS Alias can be used for resources
    - No health checks
  - `Weighted`
    - Add a value in %
    - Weight define how much traffic will be used for that record
    - Can be used with health checks
    - Can be used to test new app 
  - `Latency`
    - Redirect to lowest latency close to us
    - Latency is calculated between user and AWS Regions

### Health Checks Router53
  - Available only for public resources
  - Monitoring endpoints = App, Server, EC2, other resouce
  - Monitor other healthchecks
  - Around 15 global health checkers
  - 3 failures = unhealhty, every 30s
  - Higher frequency = higher cost
  - If 18% health checkers reports it is healthy = it is healthy
  - Passing if status code is 2XX or 3XX
  - Health check can be also based on text file = first 5120 bytes
  - Security groups/Firewall must allow access from healthcheckers
  - `Calculated Healthchecks` - combine multiple regular healthchecks, can use 'OR','AND' or 'NOT'
  - Private subnets can be monitored through cloudwatch metrics, first monitor private segment via cloudwatch ...
  - Add alarm to cloudwatch and then healthcheck can monitor the alarm

### Geolocation
  - Routing policy
  - Specify A receords based on locations
  - Example if someone is from EU they will get record A from Frankfurt EC2 IP address

### Geoproximity
  - Routing policy
  - Based on geo location
  - Specify Bias (1,99 = more traffic / -1,-99 = less traffic)
  - Bias is prefference value
  - if one location has higher bias, then it will get more traffic
  - Traffic policy is UI which via GUI helps to create complex rules
  - GUI is great for geoproximity as it have a map and shows who will connect where

### MultiValue
  - Used to send traffic to multiple IPs for same A record
  - Can be associated with HealhChecks
  - Up to 8 healthy records
  - Not replacement for ELB
  - Create multiple records with the same A record but different IP
  - Better then regular mutltivalue as it does not support healthchecks


## Amazon S3 - Simple Storage Service
  - Infinite scalling storage
  - Very popular
  - Using objects and buckets
  - Objects are data and have key
  - The key is the full path to data, example: s3://my-bucket/folder/folder2/file.txt
  - Buckets are main folders for data and can contain sub folders
  - Buckets are regional scoped
  - It must be unique name (for full amazon)
  - Max size is 5TB for 1 object
  - Uploading bigger files are multi-part upload
  - Metadata - like tag - key/value
  - Objects contains versions

### S3 Versioning
  - Enable per bucket level
  - It keeps multiple archives of same data during overwrite
  - Helps during unintended overwrite to rollback to previous version (like git)
  - In bucket setting we can enable versioning later on too
  - Delete files are there with Delete marker
  - We can revert it
  - Deleting file with delete marker is permanent delete

### S3 Encryption
  - 