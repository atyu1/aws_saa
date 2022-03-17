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

## Advanced IAM
### AWS Security Token Service (STS)
  - token to grant a temporary and limited access to AWS resources
  - AssumeRole
    - Within your account
    - Cross account
  - AssumeRoleWithSAML
    - return credentials with users logged with SAML
  - AssumeRoleWithWebIdentity
    - Used with Cognito - logged via FB,Google, ...
  - GetSessionToken
    - for MFA, from user or root account
  - SAML 2.0 Federation = Auhenticate within org, get local token, used to authenticate to AWS, get STS and access the AWS resource
  
  - UseCase:
    - Login from App to Google
    - Use that token to login to cognito
    - Cognito verify the Google Token and attach STS to have access to S3
    - User can access with new token the S3 for limited time

### AWS Organizations
  - Global Service
  - Allow to manage multiple org
  - The main account is the master 
  - Other accounts are member of specific organizations
  - Possible to use consolidated billing
  - Pricing benefits as we consolidate and can hit lower price tiers
  - Create per department or dev/test/prod or multiple sub customers
  - Use tagging for billing
  - Cloudtrails to track user activities
  - Master Account -> Multiple OU -> Multiple Accounts or OU again and have accounts ...
  - `Service Control Policies` - whitelist or blacklist IAM actions
  - Use case: restrict access to services or PCI compliance
  - Master account does not apply to Master Account

  - `Migrate Account`
    - Remove member from organization
    - Send invite to the new one
    - Accept invite

### AWS RAM
  - Resource Access Manager
  - Allow to share resources with other accounts
  - Can share within organization or outside
  - Used mostly with VPC or Route53

### IAM Conditions
  - Those are conditions inside Policies
  - awsSourceIp = Source IP address for API calls
  - awsRestrictedRegion = restrict the API calls `to` specific region
  - ec2ResourceTag/<key>:<value> = restrict based on tags for EC2
  - awsPrincipalTag/<key>:<value> = restrict based on tags for service
  - 

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

### Global Aurora
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

## AWS Redshift
  - PSQL based but used for online analytics processing - business analytics
  - 10x better performance and scale up to PB 
  - Columnar storage data (like cassandra)
  - Massive parralel queries
  - Pay as you go
  - Data is loaded from other DBs: Dynamo, S3, ...
  - Creates nodes, 1-128
  - 1 Primary for Query and planning
  - rest are computes to distribute the query and have better performance
  - Backup/Restore
  - Perform queries against S3 directly, no need to load
  - All in 1 AZ
  - Use snapshot to copy to another AZ/Region
  - Snapshot can be automated or manual
  - Ingest data:
    - Kinesis Firehouse
    - S3 copy command (via internet or via VPC (via enhanced VPC routing))
    - EC2 directly push - JDBC driver
  - Faster then Athena
  - Cost - pey per provisioned nodes

## AWS Glue
  - Managed extract, transform and load service (ETL)
  - Used for analytics to process the data first
  - Glue catalog - tables which have same format after data load
  - Glue data crawler - can load data to catalog

## AWS Neptune
  - Fully managed graph DB
  - Used in relation search, social networks
  - Highly available, 3 AZ
  - 15 read replicas 
  - Classic security: IAM,KMS,SSL

## AWS ElasticSearch (OpenSearch - new name)
  - DB created for fast searches
  - Search not just for Primary Keys or Indexes
  - Support parial search
  - Complement to another DB
  - Built-in integration: Kinesis, Firehouse, IoT, CloudWatch, ...
  - Security: IAM,KMS,SSL + Cognito
  - Mostly come as ELK stack

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
  - Files bigger then 5GB must be uploaded as multipart
  - Explicit DENY in an IAM Policy will take precedence over an S3 bucket policy.

### S3 Versioning
  - Enable per bucket level or per file during upload
  - It keeps multiple archives of same data during overwrite
  - Helps during unintended overwrite to rollback to previous version (like git)
  - In bucket setting we can enable versioning later on too
  - Delete files are there with Delete marker
  - We can revert it
  - Deleting file with delete marker is permanent delete

### S3 Encryption
  - 4 methods to encrypt
  - SSE-S3
    - Encrypts S3 objects using keys managed by AWS
    - Server Side Encryption (SSE)
    - AES-256
    - Must set header: "x-amz-server-side-encryption":"AES256"
    - HTTP or HTTPS used to upload the object with header and it will know that it must encrypt
  - SSE-KMS
    - using AWS KMS to manage keys
    - Same as S3 but with custom keys with KMS 
    - Must set header: "x-amz-server-side-encryption":"aws:kms"
  - SSE-C
    - using own keys outside of AWS 
    - HTTPS must be used (for secret)
    - encryption key is stored on 3rd party server, not on amazon
    - Data key is located on header
    - For decrypt we need to also provide the key
  - Client Side Encryption
    - Clients encrypt before data upload the object
    - CLients decrypt if they download the data 
    - Amazon is not involved in any encryption/decryption
  
  - Clients and API should use HTTPS - encryption in transit to use SSL/TLS
  - Default encryption if enabled, used for every non encryption configured objects
  - Object encryption config overwrites default

### S3 Security
  - User Based
    - IAM Policies are leveraged
    - We can make it granular to specify what is allowed

  - Resource Based
    - Bucket Policies - rules applied for full bucket (cross account)
    - Object ACL - more specific
    - Bucket ACL - less used

  - IAM user must have access and Resource ACL must allow to have successsful access to the data
  - `S3 Bucket Policies`
    - Json based
    - Example (like earlier in IAM policies):
    ```
{
  "Version": "2020-01-01",
  "Statement":[
    {
      "Sid":"PublicRead",
      "Effect":"Allow",
      "Principal":"*"m
      "Action":[
        "s3:GetObject"
      ],
      "Resource":[
        "arn:aws:s3:::mybucket/*
      ]
    }
  ]
}
    ```
    - This example shows how to provide read access for every object in bucket for everyone 
    - Block public access to buckets and its objects is possible per bucket
  - `Other Security`
    - Networking - supports VPC endpoints
    - Logging and Audit - S3 access logs can be stored on S3 bucket, API calls logged via AWS Cloud Trails
    - User Security - Use MFA for delete (possible to enable just for deletes) & Pre-Signed URL (limited time to be accessible)

### S3 Websites
  - Static websites can be hosted in S3
  - Bucket policy must be public and allow access otherwise 403 will be returned
  - Static website hosting must be enabled per bucket

### CORS
  - Cross Origin Resource Sharing
  - Origin = scheme (protocol), host and port, like http://test.com
  - Browser blocks cross origin access if no special headers are provided
  - Header: Access-Control-Allow-Origin: <URL>
  - We need it if we have a website in one bucket and some files in another bucket
  - Cross Origin = Cross Bucket access
  - Limit per bucket or "*" for all
  - Cross Origin must be enabled on second bucket, not on main
  - Possible to debug and see in console of browser
  - In GUI, in CORS section, add in JSON format the headers

### S3 MFA Delete
  - For deleting objects, you have to use MFA
  - bucket owner can enable/disable it
  - only via CLI can be enabled
  - To enable:
    - User must have MFA
    - S3 versioning enabled
  - MFA will be required for suspend versioning or permanent delete
  - Version list is not MFA required or enable versioning

### S3 Access Logs
  - Every request to S3 from any account will be logged into separate S3 bucket
  - Data can be used for analysis or audit
  - Monitored bucket cannot be the logging bucket! (LOOP)
  - There is special field in properties -> Server Access Logging -> Enable (Specify the log bucket)
  - Automatically modify the ACL for source bucket
  - It creates folders for logs in bucket

### S3 Replication
  - Versioning must be enabled
  - 2 Types:
    - CRR - Cross Region Replication
    - SRR - Same Region replication
  - Can be under different account
  - Copy is async
  - IAM permissions are required to S3
  - Usage:
    - CRR - complaince, lower latency, replicate to multiple accounts
    - SRR - live replication between multiple accunts for analysis
  - After enable, only new objects are replicated
  - Old objects must be copied manualy
  - Replicated objects cannot be re-replicated to another bucket
  - Deleted objects can be optionaly deleted on replicated bucket, if enabled
  - Deleted objects for specific version are not replicated

### S3 Pre-Signed URL
  - Generate pre-signed URLs using SDK or CLI
  - Use for non public buckets
  - Downloads - can use CLI
  - Uploads - harder, use SDK
  - by default valid for 3600 sec (change by --expires-in)
  - users using the URL will have permission of user generated the URL for GET/PUT
  - CLI: aws s3 presign <s3://bucket/object> --region <region> --expires-in <timme>

### S3 Storage Classes
  - `S3 Standard`
    - General Purpose
    - High Durability of objects across Multiple AZ
    - AZ disaster resilient
    - Price: 0.023/GB
  - `S3 Standard Infrequent Access (IA)`
    - Data less frequently access but needs rapid access if neeed 
    - Multi AZ
    - High durability 
    - Lower cost as Standard
    - Usage: Backups, less frequent accessed files
    - Price: 0.0125 / GB
  - `S3 One Zone IA`
    - Object stored on single AZ
    - Same durability withing AZ
    - Low latency
    - Low cost (20% less then IA)
    - Usage: secondary backups, storing redundant data
    - Price: 0.01 / GB
  - `S3 Inteligent Tiering`
    - Same performance as Standard
    - Monthly monitors the objects and auto-tier them based on access
    - It can move files to  IA or Glacier
    - Price: 0.0125-0.023 / GB + Cost for monitoring = 0.0025 / 1000 Objects
  - `S3 Glacier`
    - Archive
    - Low cost for long archiving, data saved for 10years and so
    - Cost per month 0.004USD/GB + retrieve price
    - Objects = called Archive
    - Buckets = called Vautls
    - Retrieve options:
      - Expedite = 1-5 minutes 
      - Standard = 3-5 hours
      - Bulk = 5-122 hours
    - Faster retrieve = higher price
    - Minimum storage duration 90days
    - Price: 0.004 / GB
  - `S3 Glacier Deep Archive`
    - Extra long term storage (like tapes)
    - Cheaper then Glacier
    - Retrieve options:
      - Standard = 12 hours
      - Bulk = 48 hours
    - Minimum 180 days
    - Price: 0.00099 / GB

### S3 Lifecycle Rules
  - Transition Actions: Define, when the storage can be moved to another storage class
    - Example: Move object to Standard IA after 60 days, Move to Glacier after 6 months
  - Expiration Actions: configure, when to delete , After 1 year delete the files
  - Rules can be applied to limit the folders in buckets, prefix 
  - Example: Move current version to Standard IA after 40 days, delete versions after 60 days, ...

### S3 Analytics
  - Help to determine when to move objects to Standard IA
  - Non usable with Onezone_IA or Glacier
  - Daily refresh
  - First start visible after 24-48h
  - Create report which can help you to create Lifecycle Rules

### S3 Performance
  - Scaled to high request rates - 100-200 ms
  - Application benchmarks:
    - 3500 requests per second per prefix- PUT/COPY/POST/DELETE
    - 5500 requests per second per prefix- GET/HEAD
  - No limit to number of prefixes
  - Prefix: name of the folders and subfolders (all between bucket and file): Example: bucket/folder1/folder2/file -> folder1/folder2 = prefix

  - KMS Limitation - SSE-KMS if used can limit the performance, upload/download  do encrypt/decrypt
    - KMS have quota 5500, 10000, 30000 req/s
    - It can be increased in Service Quota Console
  
  - Optimization: (Upload)
    - Multi-Part upload, makes upload to smaller chunks in parralel, recommended >100MB and must >5GB, amazon joins the parts and create file
    - S3 Transfer Acccelaration: transfer files to edge locations, that will forward data to S3 bucket, can be used with multipart upload
  
  - Optimization: (Download)
    - Paralelyze GET for byte ranges, better resilience, big files don't download directly

### S3 Select / Glacier Select
  - Retrieve less data using SQL filtering
  - Filter by basic SQL parameters (rows, columnds)
  - Better CPU on client side and less network traffic

### S3 Event Notifications
  - Events: Object created, deleted, move, restored
  - Filtering by patterns (*)
  - Usage Example: S3 generate a thumbnails of images uploaded to S3
  - S3 can create many S3 events
  - Events are generated in seconds, but sometimes delay may happen to minutes
  - 2x writes at the same time to single object can generate only 1x event
  - Choosable where to send notifications: SQS, SNS or AWS Lambda

### S3 Requested Pays
  - In general bucket owners pays for transfer costs
  - If enabled, requestor user will pay for downloads and networking costs
  - Helpful when to share big data
  - Client must be logged (cannot be anonymous)

### S3 Glacier Vault Lock
  - Write Once Read Many
  - Lock the policy for 
  - Compliance ensure that data is not modifiable
  - `S3 Object Lock`
    - WORM model 
    - Block an object version deletion for specified time
    - Object retention, specify period or legal hold (no expiration date)
    - Modes:
      - Govermance mode - users cannot alter or delete objects unless special permission
      - Compliance mode - user cannot alter ot delete object, even root cannot do, retention period cannot be changed or shortened

## Athena
  - Serverless query service
  - Performs analytics for S3 objects
  - Using stanadard SQL queries
  - Support various file formats: JSON, CSV, ORC, ...
  - Price: 5$ per TB for data scanned
  - Use compression for less scan = lower cost
  - Usacase: Business Inteligence, Analytics, Reporting
  - Exam tip: serverless SQL for analytics = Athena

## AWS CloudFront
  - CDN
  - Improves the read performance by caching at the edge
  - Edge locations are connect to main AWS DC by fast private networks
  - Plenty of edge locations (check page)
  - DDOS protection, integrated with AWS Shield and AWS Web Application FW
  - Used for:
    - S3 bucket - distribution of file (without making it public), caching at the edge, upload, enhanced security with CloudFront Origin Access Identity
    - Custom HTTP - application load balance, EC2 instance, S3 website 
  
  - If EC2 is used, we must allow edge location IP in security groups, located in website
  - Same for ELB for internet frontend side
  - GEO restrictions - can limit countries to access the access - whitelist or blacklist
  - Pricing depends on locations, also more data moved over = less cost
  - Price classes:
    - All - all regions best perfomance - costly
    - 200 - most regions, excludes most expensive
    - 100 - limited, only the least expensive
  - Possible to do Application definition per URL
  - Different contend (like /images or /api) can be routed to different origins (ELB, EC2, S3, ...)
  - We can add secondary origin for errored issues

  - Field Level encryption - protect sensitive data throuhg application stack, adds more security
    - Sensitive information encrypted closer to the edge
    - Using Asymetric encryption
    - Usage: specify set of fields in POST which should be encrypted

### AWS Cloudfront Signer URL / Signed Cookies
  - Example: we can give access to premium content for paid users
  - Use CloudFront Signed URL/Cookie
  - It can include URL Expiration, IP ranges to access data, trusted tiers
  - Public things should be short lived (music) but sharring for specific user can be longer, last for years
  - Similair to S3 signed url but it allow access no matter of origin

## AWS Global Accelerator
  - Using anycast IP to reach the closest AWS endpoint and after get resource via internal AWS network
  - Resource can be Elastic IP, ELB, ALB, EC2, ...
  - Does not cache
  - Anycast are used to reach the closest frontdoor
  - Anycast IP is static
  - Better to be used for gaming, non http or http but non cachable traffic

## AWS Snow Family
  - Used to:
    - Collect and Process data at the Edge
    - Migrate data into and out of AWS
  - Products for Data Migration:
    - Snowcone
    - Snaball Edge
    - Snowmobile
  - Product for Edge Computing
    - Snowcone
    - Snoball Edge
  - Using it if we don't wanna transfer huge data over internet
  - Offline devices where we copy and manualy move data, which are loaded to cloud/S3
  - Snowball can import to S3 Standard only (after lifecycle policy can move to Glacier)

### AWS Snowball Edge
  - Physical data transport
  - Move TB or PBs of data
  - Pay per Data transfer
  - Using Block storage which is S3 compatible
  - 2 types:
    - Snowball Edge Storage Optimized - 80TB
    - Snowball Edge Compute Optimized - 42TB
  - Edge computing is places which generate data but no internet connectivty, it will process and store
 
  - `Compute`
  - 52vCPUs, 208GB RAM
  - Optional GPU (Machine learning)

### AWS Snowcone 
  - Physical data transport
  - Smaller then Snowball Edge and Light 1,2 Kg
  - 8TB of storage
  - Own battery is needed
  - Can be sent back to AWS offline or use AWS DataSync to send data

  - `Compute`
  - 2CPUs, 4GB of memory, wired or wirelless connection
  - USB-C

### AWS Snowmobile
  - Real Truck
  - Each have 100PB of capacity
  - Physical security (GPS, surveilance, temperature control, 24/7 watched)
  - More can be transfer by ordering more trucks

### AWS OpsHub
  - Alternate to CLI
  - Software to download and manage Snow Family by GUI

## AWS FSx
  - Launch 3rd Party High Performance FS on AWS
  - AWS Fully Managed
  - Usage:
    - Lustre (Large scale computing FS ) - Linux and Cluster = Luster, Machine Learning, High Performance Computing HPC, Video processing, ...
    - Windows File Server (SMB and Windows NTFS, Used for windows) - data backed up daily to S3
    - NetApp ONTAP
  - 2 types of FS:
    - Scratch FS - High Burst, Temporary storage (data not replicated), Usage: short term processing
    - Persistent FS - Long term storage - replicated within AZ, Usage: long term processing

## AWS Storage Gateway
  - Used in hybrid cloud
  - Bridge between local data and AWS in S3
  - GW is located on premise
  - Data from any GW are saved to S3 on AWS
  - 3 types of GW:
    - `File GW`
      - S3 bucket using NFS pr SMB protocol
      - Supports S3 (standard, IA, One Zone IA)
      - Bucket Access using IAM for each File GW
      - Used Data is cached in GW
      - Can be mounted to many server on premise
      - Support AD integration (Active Directory) for Auth
    - `Volume GW`
      - Using iSCSI backed by S3
      - Supported snapshots
      - Cached Volumes: low latency access
      - Stored Volumes: dataset on premise and scheduled backups to AWS S3
    - `Tape GW`
      - Virtual Tape Library backed to S3 or Glacier
      - Works with most common tape vendors
      - using iSCSI too
  - Storage GW Appliance - Box provided by AWS, if there is no free server for customer
  - Have proper resources (CPU,Memory, ...) for any kind of GW

## AWS FSx File Gateway
  - Native Access for Windows File Server
  - GW located on CU premise
  - Important feature: Cache for frequently accessed files/data
  - Useful for Group file shares or Home directories

## AWS Transfer Family
  - Using FTP,FTPs,SFTP 
  - Managed by AWS
  - Used as alternate copy/access to AWS Data
  - Pay per hour and transfer per GB
  - Store and Manage user crednetials
  - Integrate with various Authentication servers

## AWS Standard Queue System (SQS)
  - Producer - create messages / sends to Queue
  - Queue - stores the messages
  - Consumer - process the messages / poll from Queue
  - Fully managed Service
  - `Decouple Applications`
  - Unlimited number of messages
  - 4days TTL for message by default
  - max 14days
  - Low lantency (<10ms)
  - Max size is 256KB per message
  - Supports duplicate messages and out of order

  - `Producer`
    - Produce messages using SDK to send to SQS 
    - Using API: SendMessage()
    - Message stays in SQS until consumer reads and delete it

  - `Consumers`
    - Can be EC2, custom server or AWS Lambda
    - Poll SQS for messages 
    - Processing it and when all went fine, delete it from Queue
    - Delete API: DeleteMessage
    - Multiple consumers can read messages from the same SQS
  
  - Great Usacase: SQS with ASG
    - EC2 instances inside ASG polls messages
    - Cloudwatch can see the Queue Length (number of messages)
    - Alarm if queueue length reach the limit
    - CloudWatch Alarm can inform ASG to Scale Out
  
  - `Security`
    - Supports HTTPs
    - Data at rest encrypted by KMS
    - Client side encryption/decryption
    - IAM policies to limit access to SQS API
    - SQS Access Policies - similair to S3 bucket policies
  
### AWS SQS Access Policy
  - Allow to define more granular access
  - Like Prinicpal can just receive message from specific Queue
  - With SQS source policy, we can enable to send message to SQS if object is uploaded

### AWS SQS Message Visibility Timeout
  - if consumer polls the messages, it will be invisible for other consumers (after ReceiveMessage())
  - the message will be invisible till "message visibility timeout"
  - by default 30s
  - after if message is not deleted it will be visible again
  - if consumer needs more time to be processed, there is API: changeMessageVisibility to modify the  timer
  - if timer is too short, it can create duplicate data after processing

### AWS SQS Dead Letter Queues
  - if message goes back often to the Queue, we can set maximum receive threshold
  - After max, we can send the suspicious message to send a dead letter queue for analysis
  - it is good for debugging
  - DLQ should have high retention time - 14days

### AWS SQS Delay Queues
  - Consumer don't see the message immediately
  - We can do full per queue or per message value
  - In GUI, we can setup with `Delivery Delay` field

### AWS SQS Long Polling
  - Consumer can poll longer time the Queue, if Queue is empty
  - It decrease the number of requests and we have lower latency
  - 20s is max (prefferable)
  - We an enable per Queue or per API
  - Per message it is API: WaitTimeSeconds

### AWS SQS Request-Response Implementation
  - Decouple request messages to one Queue and response from consumers to another Queue
  - It will make processing faster and more effective
  - To which Queue to response, we can setup by: Reply To: "response queue name"

### AWS SQS FIFO Queue
  - Ensure ordering of messages in Queue and conumer process in order
  - Limited throughput (300msg/s) or with beching we have 3000msg/s

## AWS Simple Notification Service (SNS)
  - The service sends messages to SNS
  - SNS have subscribers and get notifications if message
  - messages are broadcast to every subcribers
  - its called pub/sub 
  - we have 100000 subcriptions per topic
  - topic = queue
  - subscribes = consumers (passive, not polling)
  - Subscribers can be:
    - SQS
    - HTTP/HTTPS
    - Lambda
    - Emails
    - SMS message
    - Mobile Notifications
  - Many AWS Service can send data to sns, like cloudwatch (alarms)
  - We can pulblish by topic:
    - Use SDK
    - Create topic
    - Create subscriptions
    - Publish to topic
  
  - Supports FIFO like SQS 
  
### AWS SNS Security
  - Same as SQS
  - HTTPS
  - KMS keys for data encryptions
  - client side encyrption decrypton
  - IAM policies for Access contorler
  - SNS Access Polisies 

### AWS SNS with SNS
  - SNS can be used as front queue 
  - and multiple SQS queues can subscribe
  - push once and receive on many
  - its scalable in future
  - S3 sends event only 1x so its agood use case

### AWS SNS Message filtering
  - JSON policy to filter messages sent ot SNS
  - without filter, allow all messages
  - its filtering which messages are sent to which subscribers
  - so it can be instead of broadcast a multicast
  - Filtering can be done by values in messages

## AWS Kinesis
  - Makes streaming data processing easier
  - Hpels with collect, process and analysis
  - Works with real time data
  - Pay for consumption rate
  - Data can be: Application Logs, Metrics, Website clicks, IoT telemetry, ...
  - 2 services:
    - Kinesis Data Stream - capture, proces and store data streams
    - Kinesis Data Firehouse - load data streams into AWS data stores (not custom consumers, can be 3rd party, Redshift, ..)
    - Kinesis Data Analytics - analyze streams with SQL or apache flink
    - Kinesis Video Stream - capture, process and store video streams

## AWS MQ
  - Used as old classic MQTP protocols
  - Good to move old apps without rewrite the code
  - Using Apache ActiveMQ
  - On exam, if question is about moving app to cloud and don't refractor the code to SQS then use MQ

## AWS Elastic Container Service (ECS)
  - Launch Docker containers on AWS
  - You must provision and maintain the infra for them
  - AWS will take care of start/stop of containers
  - We handle EC2 instances with special AMI - this contains ECS Agent
  - EC2 instances have ECS agent - to handle the containers
  - We need to assign IAM Roles 
  - EC2 Instance Profile
    - Used by ECS agent
    - We assing IAM roles so it can communicate with other AWS services
    - We use for, make API calls to ECS, send logs to CloudWatch logs, Pull docker image from ECR, refference sensitive data 
  - ECS Task Role:
    - Attach a specific Role to a task
    - It will limit the access of containers to specific AWS services
  - EFS is mostly use to share file system and data between tasks

### AWS ECS Services & Tasks
  - Service is a special purpose tasks , accross multiple EC2
  - We can attach ALB to Containers/Tasks
  - It will support automatic port mapping
  - EC2 must have security group to allow all port from ALB

## AWS Fargate
  - No EC2 management is required, all automatic
  - Serverless offering
  - AWS will just run containers for you
  - Every task have own ENI to access the container via IP 

  - Load balancing can use ALB too
    - Enable to have unique IP via ENI
    - ENI security group must allow traffic from ALB

### AWS ESC Scaling
  - We can setup a cloudwatch metric to watch CPU utilication
  - If its high, create new cloudwatch alarm and this will inform autoscaling groupt to run a new task
  - Optional: Use ECS Capacity Providers - it will monitor if EC2 have enough resource and add more EC2 if needed
  - Another option is to use SQS before Tasks and scale besed on SQS the number of tasks

### AWS ECS Rolling Updates
  - We can control Minimum healthy and Maximum percent
  - When update happen, it will keep minimum old versions defined running
  - The rest will be terminated and updated
  - When those are stable, the old one at the beginning will be updated
  - Maximum define additional tasks to create only during upgrade

## AWS Serverless
  - You don't manage infra / servers
  - Only deploy code
  - Function as a Service
  - We leverage all AWS services and do most of logic in lambda

### AWS Lambda
  - No infra to manage
  - Easy pricing - have free tier
  - Pay per request and compute time
  - Integrated with whole AWS service and many programming languages
  - Monitoring through AWS CloudWatch
  - Limits per region - Execution
    - 128MB - 10GB Memory
    - Max execution time: 900 second (15min = timeout)
    - Env variables max: 4kB
    - Disk capacity max 512MB
    - Concurency execution: 1000
  - Limits per region - Deployment
    - Max size: 50MB compressed /250MB non-compressed
    - /tmp can be used to load bigger files
    - Size env variables to 4kB

### AWS Lambda@Edge
  - Using CloudFront - AWS Lambda can filter before app
  - Build better/responsive apps and no need for server
  - It can customize CDN content
  - Pay only for usage
  - Used to change Request or Response on CloudFront

## AWS DynamoDB
  - Fully Managed DB, Highly Avaliable and Replicated accross multiple AZ
  - NoSQL DB
  - Scales to massive workloads, distributed DB
  - Milions of requests per second, trilions of row, 100s of TB of storage
  - Low latency
  - Made of tables
  - Each table have primary key and optionally sort keys
  - Other fields are called attrobutes
  - Each table can have infinite number of items
  - Provisioned Mode (default)
    - Specify number of Reads/Writes per second
    - Plan capacity before hand
    - Pay for provisioned Read/Write Capacity Units (RCU & WCU)
    - Possible to add auto-scalling mode for RCU/WCU
  - On Demand Mode
    - Read / Writes are automatically scalled
    - No capacity planned
    - Pay for what used but more expensive
    - Great for unpredicated data
  - Max size for Item in DB: 400kB

### AWS DynamoDB DAX
  - Fully managed, HA, cache for DynanmoDB
  - Helps with Reads
  - Microseconds for latency
  - No app logic is required

### AWS DyanmoDB Streams
  - Every CUD can generate stream
  - Stream can be processed by Kinses or other services

### AWS DynamoDB Global Table
  - Table accessible in multiple regions
  - Same table seen and replicated bwtween regions
  - Read/Write can be to table in any region
  - Prerequisite: Streams

## API Gateway
  - Good integration with Lambda for serverless application
  - Support Websocket protocol
  - Handle API versions, environment (dev, prod), security
  - Create API keys and handle throtling
  - Swagger/OpenAPI import to quickly configure
  - Generate SDK and API spec
  - Cache API responses
  - 3 types:
    - Edge-Optimized - global clients, request routed through CloudFron Edge locations (improve latency)
    - Regional - for clients within a region
    - Private - accessible only from private VPC

### API Gateway Security
  - IAM permissions
    - Attach IAM policy to users
    - Add access within your org
    - Leverage "sig v4" capability where IAM credentails are in headers

  - Lambda Authorizer = Customer Authorizer
    - used for 3rd party tokens
    - Validate tokens in header
    - Option to cache 
    - Helps with OAuth, SAML or 3rd party Auth
    - Lambda returns IAM policy for user

  - Cognito
    - Manage user lifecycle
    - API GW verifies user identity via cognito
    - No custom implementation
    - Only used for authentication

## AWS Cognito
  - We use for authentication
  - Cognito User Pools
    - Sign in fuctionality for app users
    - Used with API GW
    - Serverless database of users
    - Can enable federated identities (FB, Google, ...)
    - Sends back a JWT Token
  - Cognito Identity Pools
    - Provie AWS credentials so access via AWS identity
    - Integrate User Pools as integrity provider
    - Federated Identiy Pools - Helps to get AWS resources via 3rd party auth - like access to S3 with Facebook login
    - Allows to comminucate with AWS Services through App users
  - Cognito Sync
    - Synchronize data from device to Cognito
    - Deprecated - AppSync
    - Store configs and other info

  - Authenticate from mobile via APP to S3: Use Cognite -> Generate AWS STS -> Access to S3 via STS

## AWS SAM
  - Serverless Application Model
  - Framework for deploying and developing servrless apps
  - All config is YAML code
  - something like Infra as a Code
  - run Lambda, API GW or DynamoDB locally
  - use CodeDeploy to deploy lambda functions

## AWS CloudWatch
  - Provides metrics to every service
  - Metric is variable to monitor - CPU, Network, Memory, ...
  - Every metric is part of namespace
  - Dimension - additional fields/attributes of metrics, up to 10x per metric
  - Metric have timestamp
  - CloudWatch support Dashboard of metrics
  - Detailed monitoring - mor often -> high cost
  - Detailed monitoring supports custom metrics
  - Custom metrics
    - Can be pushed by API - PutMetricData
    - Can add dimensions (Environment.name, Instance.id)
    - Must have "StorageResolution" API = Standard 1min, High Resolution 1,5,10,30sec (higher cost)
    - Timestamp is not verified, any value in future or past is accepted

### AWS CloudWatch Dasbhoards
  - Quick way to see metrics and alarms
  - Global service
  - Can include graphs from different account or regions
  - Change TZ or timerange, automatic refresh
  - Can be made public by 3rd party SSO as well (Cognito)
  - Similair to Grafana
  - Pricing:
    - 3 dashboards free (50 metrics)
    - $3 per dashboard per month

### AWS CloudWatch Logs
  - Similair to logstash
  - Log group - name of logs comming from source, custom name
  - Log stream - instances whith applications / log files or containers
  - Support log expiration
  - Can send logs to S3, Kinsesis, Lambda, ElasticSearch
  - Sources: SDK, Beanstalk, ECS, Lambda, VPF FLow logs, API GW, CloudTrail, Route53
  - To push to CloudWatch from EC2, we need agent
  - 2 types:
    - CloudWatch Logs Agent - old, can send logs only
    - CloudWatch Unified Agent - newer, can send logs and metrics, Use SSMP Parameter Store = centralized config for agent config

### AWS CloudWatch Alarms
  - Notification on any metrics
  - Various options: sampling, %, min, max
  - Alarm states: OK, INSUFICIENT_DATA, ALARM
  - Period: length of time to evaluate the metric
  - 3 targets:
    - EC2 - Stop,Reboot,Terminate
    - EC2 AutoScaling - trigger scale action
    - SNS - send notification to SNS

  - EC2 Recovery - keeps the same IP, setup cloudwatch -> check health status -> create alarm and do recovery -> optional: send info to SNS

### CloudWatch Events
  - Event Pattern, intercept vents from sources / schedule or Cron
  - Can watch some events in AWS and do action like send it to SQS

### AWS CloudWatch EventBridge
  - Events v2.0
  - Creates multiple buses to separate events:
    - Default - Generated by AWS services
    - Partner - recieve email from SaaS or apps
    - Custom - for your own apps
  - Can be accessed by other AWS accounts
  - Schema Registy - analyze and events and create schema
    - Create specific code, which can understand the schemas

## AWS CloudTrail
  - Provides audit logs for your AWS account
  - Complaince and governance
  - Enabled by default
  - Get history of events and API calls (API, SDK, CLI, Console)
  - Can put logs from cloutrail to cloudwatch logs or S3
  - 3 kind of events:
    - Management - logged by default, account activity, service creation, ... 
    - Data - not logged by default, S3 put/delete object, AWS Lambda execution ... (too much)
    - Insight - analyze events and provide unusual activity, continous analyzis ongoing to decide
  - Kept for 90 days, for longer keep send to S3
  - If its on S3 possible to analyze with Athena

## AWS Config
  - Used in auditing and compliance for AWS resources
  - Helps record config changes
  - Its a regional service - but can be aggragated accross accounts
  - Possible to send SNS when config changes
  - Possible to send to S3 and analyze with Athena
  - Rules add possibility to check if complainent
  - AWS config notifications can send update if needed

## AWS Single-Sign On (SSO)
  - Free service
  - Used to login to once and reuse the same login to multiple sservices
  - Can be used within AWS like between Organizations
  - Or to 3rd party services like dropbox, slack, office 365, ...
  - Uses SAML 2.0 markup
  - Integrated to ActiveDirectory
  - Centralized permission management and auditing

## VPC
  - Default VPC always exists per account
  - EC2 by default use default VPC - can be changed by using another subnet
  - Deafult VPC is public
  - Creates subnets for Every AZ in that region
  - Attach the subnet to Internet GW so we have public access
  - Max 5 per region (Can be increased)
  - Max 5 CIDR - between /16 - /28
  - Only private IPs are allowed
  - VPC IP cannot overlap

### Subnets
  - Attach to VPC
  - AZ specific
  - 5 IPs are reserved (first 4 + 1 last)
    - Network
    - VPC Router
    - AWS mapped DNS
    - For future use
    - Broadcast

### Internet Gateway 
  - Allow VPC to connect to Internet
  - Scales horizontaly and HA and redundant
  - Must be created separately (not part of VPC creation)
  - One VPC can be attached to 1 GW and vice versa 1:1
  - After attaching we have to tune the route table to have access to internet

### NAT Instances
  - Outdated - not recommended
  - General NAT, provide access to outside from inside
  - NAT instance is just EC2 which runs in Public and Private subnet (Special AMI created by AWS)
  - Must have Elastic IP attached to it
  - Can be bottleneck if EC2 instance is higher then this box
  - No HA
  - Manage security groups on your own

### NAT Gateways
  - AWS Managed, better bandwith, HA and no administration
  - Pay per hour usage and bandwith
  - Require IGW
  - Bandiwth: 5Gbps -> scalabe to 45Gbps
  - No security groups to manage
  
### enableDnsSupport
  - If enabled, instances from public subnet VPC have access to Route 53 DNS server
  - DNS server is located at: 169.254.169.253
  - Or can be used the subnet (.2) - as mentioned second is for DNS (after network 0, gw 1, dns 2)
  - if not enabled then custom dns needs to be used

### enableDnsHostnames
  - enabled by default for default VPC and not for other VPCs
  - if True, assing public hostname to EC2 if it have public ipv4
  - this feature assign public DNS to EC2

### Security Groups
  - assigned to instance
  - stateful = reply traffic is allowed by default

### Network Access Lists (NACL)
  - attached to subnet
  - stateless - retrun traffic must be also explicitly enabled
  - NACL control traffic per subnet
  - NACL rules have priorities
  - Default NACL - accepts in or out = open
  - NACL should enabled ephemeral ports which are high range port for reply traffic for clients

### VPC Reachibilit Analyzer
  - Diagnostic tool to analyze network connectivity issues
  - It detects if SG groups are issue or pacekts is not sent or not received or so

### VPC Peering
  - Connect 2 VPCs = VPN between them
  - They must have not overlapping CIDR
  - VPC connection is point to point and not transitive = require full mesh
  - Support cross account and cross region
  - We can refference for SG on another VPC Peer
  - Add entries to route tables to make it work

### VPC Endpoints
  - allow to have private connection from EC2 to AWS Services
  - avoid using internet GW for accessing services
  - redundant and scale horizontaly
  - Route table must be modified - it does automatically
  - Support only S3 and DynamoDB

### VPC FlowLog
  - Caputer traffic for VPC,Subnet,ENI
  - Something like netflow
  - Can send to S3 or CloudWatch logs
  - Capture traffic from network managed service too: EBS, Elasticache, redshift, workshift, NATGW, Transit GW

### Site to Site VPN
  - Creates VPN between VPC and Local DC
  - Needs Virtual Private GW
    - Its AWS side located VPN concentrator
    - Possible to customize ASN number
  - Also needs Customer GW
    - Device in DC side which do VPN
    - If device have public IP then it can be used for VPN
    - If NAT-T is before, then NAT address is used for VPN peering
  - !! Route propagation is needed to work 
  - Also security group can filter traffic for EC2 after
  - VPN CloudHub - VPGW acts as hub and spoke for multiple DC connections
  - In GUI process is:
    1. Create Customer GW
    2. Create VPN GW
    3. Create Site to site VPN


### Direct Connect = DX
  - Dedicated private connection from remote network to VPC
  - Must be setup with AWS Direct Connection
  - VPG  is required
  - Data is not encrypted but can be by using VPN
  - Resiliency:
    - High - For critical workloads (2 AWS connection to VPC with 1 link, so if one goes down there is backup link)
    - Maximum - 2 AWS connections to VPC with 2 links to avoid link failures

### AWS PrivateLink
  - Connectivity between VPCs endpoints
  - Secure and scalable
  - 1000s of VPC
  - Does not need Peering,VPGW, Internet GW, NAT GW, ...
  - This allow to establsish peering between LB or ENI via private link and no need for peering

### AWS Transit GW
  - Connects multiple VPCs throuhg it
  - Can have also connected VPN (Customer GW) or direct connect GW
  - Works accross regions
  - Use route table to limit connectivity
  - Only service which supports IP Multicast
  - Using ECMP = Equal Cost Multipath Load balancing - good for Site-to-Site VPNS with multiple connections

### VPC Traffic Mirroring 
  - Allow to do SPAN for ENIs
  - Specify from ENI
  - It can send the copy traffic to ENI or load balancer  (for higher traffic, horizontal scalling)

### Dual Stack
  - We can enabled DualStack for Internet GW for IPv6
  - ipv4 cannit be disabled

### Egress Only Internet GW
  - NAT for IPv6
  - Only usable for IPv6
  - Require Route Table tunning

## Security
  - SSL = encryption in transit
  - Encryption at rest

### AWS KMS
  - Manage all the encryption keys centralized
  - Integrated to all services like EBS, S3, Redshift, SSM, ...
  - Use GUI, CLI or SDK
  - Region limited
    - If we copy snapshots between regions, we need to re-encrypt them with new keys on other side

  - Master Key Types:
    - AES256 - Symtetric encryption, used for most of the time and envelope encryption
    - RSA or ECC - Asymetric, access to private key is not possible

  - Operations:
    - Create, Rotate, Disable, Enable
    - Audit usage by CloudTrail
    - Pricing: AWS Managed = free, custom created or imported = 1$/m
  
  - Limit: max 4kb data can be encrypted per call
  - Key Policies
    - Same as S3 bucket policies or so
    - Limit access to users
    - Default policy is permisive = allow root and entire AWS to access
    - Custom policy = can limit user for access or administration
  
  - Supports automatic rotation after rotation = evefy 1 year, name stays
  - Manual rotation is supported too, it require to attach the same alias as CMK ID will change

### SSM Parameter Store
  - Secure storage for Configuration and Secrets
  - Integrated to KMS (Optional)
  - Serverless and easy to use
  - Notification supported by CloudWatch Events
  - Integration with CloudFormation
  - Can be used with Lambda too to get conig
  - Config can have multiple hiarchies like prod,dev,testing
  - In GUI, its under system manager -> Parameter store

### AWS Secrets Manager
  - Used to store secrets 
  - Can rotate secrets and force it regularly
  - Integration AWS RDS
  - Encrypted using KMS
  - Mostly used with RDS integration

### AWS Cloud HSM
  - Hardware based security
  - Generate and manage keys 
  - Tamper resistant

### AWS Shield
  - DDos attack protection
  - 2 kinds:
    - Standard - free and activated for evey user, protects from SYN/UDP floods 
    - Advanced - paid 3000/m, provides against more sophisticated attacks

### AWS WAF = Web Application Firewall
  - Application layer secuirty 
  - Deploy on ALB, API GW, CloudFront
  - Using web ACL
    - rules: IP addr, HTTP headers, HTTP body
    - Protects against SQL Injection and XSS
    - can block based on geo location
    - rules for incomming rate as DDOS protection

### AWS Firewall Manager
  - Manage rules for whole organiztaion
  - Common rules for: WAF, Shield Adv, Security Groups for ENI

### AWS Guard Duty
  - Inteligent Threat Discovery 
  - Uses machine learning and anomaly detection
  - One click to enable
  - 30 days free
  - Watches: CloudTrail logs, VPC Flow Logs, DNS Logs
  - CloudWatch Event rule - can trigger alert/email as notification

### AWS Inspector
  - Automated Security assesments only for EC2 instances
  - Analyze OS against vulnereabilities
  - Require agent installed inside VM
  - Provides a report with results
  - Can send notification to SNS

### AWS Macie
  - AWS fully managed service
  - Security and data privacy service using machine learning
  - Will analyze S3 data and notify you about PII 
  - Used to find sensitive data and report

## Disaster Recovery
  - Event which can impact company business continuity
  - RPO - recovery point objective - how often we take backup and how much data we loose during disaster = data loss
  - RTO - recovery time objective - how long it takes to recover from disaster = downtime
  - Strategies:
    - Backup and restore
      - high RPO
    - Pilot Light
      - Some core services is always running 
      - Other less important can be started 
    - Warm Standby
      - Full system is up and running
      - During disaster scale production to other regions
    - Hot Site - multisite 
      - Full production running in cloud and on premise too
      - During disaster it can fail over
      - Both regions are Active/Active

### AWS Database Migration Service (DMS)
  - Quickly and securelry migrate DBs to AWS
  - Self healing
  - Dedicate service inside EC2
  - DB source is available during migration
  - Supports:
    - Homonogenous migration - Same type: PSQL -> PSQL
    - Heterogenous migration - Diff type: PSQL -> Oracle - requires SCT
  - SCT = Schema Conversion Tool = Define source and destination data format for migration
    - Located mostly on prem, dedicated server
  - Continous Replication = Active backup of data sent to AWS DB via DMS

### AWS DataSync
  - Move large amount of data from on-Prem to AWS
  - Can sync to S3, EFS, FSx
  - Move from NAS
  - can be scheduled hourly, daily, weekly
  - 2 components:
    - DataSync Agent - running on prem
    - DataSync Service - in AWS
  - Can be also use to sync data between regions located on EFS

### AWS Backup
  - Scheduled backup service
  - Define how often take it and how long keep it
  - Backup from various AWS resources
  - Storing backup to S3

## AWS Trusted Advisor
  - Used to provide recomendation about cloud
  - Free tier and advanced tier for more analysis

## Links:
1. Architecting for the cloud: https://d1.awsstatic.com/whitepapers/AWS_Cloud_Best_Practices.pdf (Archived)

2. Whitepapers related to well-architected framework are mentioned here: https://aws.amazon.com/blogs/aws/aws-well-architected-framework-updated-white-papers-tools-and-best-practices/

3. Disaster recovery whitepaper: https://d1.awsstatic.com/whitepapers/aws-disaster-recovery.pdf (Archived)

AWS now recommends a well-architected framework whitepaper: https://d1.awsstatic.com/whitepapers/architecture/AWS_Well-Architected_Framework.pdf