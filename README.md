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
