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
