## Things I mostly forget

- Route 53 - Geoproximity = bias (border percantage)
- RDS - 5 read replicas max
- Aurora - 15 read  replicas max
- S3 - max object size is 5TB
- S3 - max object upload size is 5GB... bigger use multi part upload
- S3 - delete marker in versioning is having own ID and 0B, if deleted files are restored
- S3 - SSE-C is only CLI or API, not console usage
- S3 - access to object for limited time = pre-signed URL
- S3 - Public access - can be enabled/disabled per account too, so all buckets will have the same status
- S3 - Static Web - Error 403 = wrong bucket policy and bucket should be public
- S3 CORS - destination page/bucket should have filtering enabled, that tell which origin can access it (so not everybody)
- Explicit Deny in IAM Policy will take presedence over Bucket Policy
- Metada for EC2: http://169.254.169.254/latest/meta-data
- S3 can send events: SNS, SQS, Lambda, EventBridge
- EventBridge - central place to collect various events and send to to services
- S3 Requester Pay - download is charged for data downloader and not owner - authenticated AWS requested account must be
- S3 Lock or Glacier Vault Lock - used to write once and lock for compliance
- Cloudfront OAI is usable only with S3 
- Cloudfront support HTTP and S3 origins
- Cloudfront Signed URL - used per file , Signed Cookie - used for many files
- Cloudfront price classes to reduce cost to limit regions - types: all,200,100 
- AWS Global Accelerator - reach closes anycast  IP for Edge Location - after reach you ALB
- AWS Global Accelerator provides 2 anycast IP
- AWS Cloudfront & Global accelerator - both use Shield to protect against DDoS
- Global Accelerator - don't cache but improves performance for TCP, UDP, IoT MQTT, VoIP or HTTP traffic with need for static IP
- !! Snowball cannot import to Glacier -> use standard and lifecycle policies
- FSx for Lustre - Seamless integration to S3, can access it as files 
- Storage GateWay is VM - Storage GW HW Appliance is  Dell server
- Amazon FSx File Gateway - Like Storage GW but specialized for NFS/Samba, supports local cache
- SQS - Fifo queues must end name .fifo
- AWS CloudFormation StackSet extends the functionality of stacks by enabling you to create, update, or delete stacks across multiple accounts and regions
- AWS SWF vs Step Functions = winner step functions
- S3 Events Fan out - send S3 event notific to SNS and it can deliver to multiple SQS for various processing
- Kinesis using shards (1MB/2MB 1000 requests/s) to store process data - partition key can be used to save to specific shard
- FIFO SQS can use Group ID to sepcify which consumers can use which group (ordering in SQS FIFO with multiple consumers)
- DynamoDB - Encrypte by default AWS OWNED CMK
- Amazon RDS simply flips the canonical name record (CNAME) for your DB instance to point at the standby
- S3 Events can be sent to SQS standard queue only
- Dynamo DB have TTL - this can expire any row after a time and delete it
- Dynamo DB Indexes - allow to search data which are attributes
- ML - Kendra (text learning), Comprehend (Natural Language Processing), Personalize (recommendations for products), Forecast, Translate, Polly (text to audio), Transcribe, Rekognition, SageMaker (custom ML)
- Disaster Recovery - Backup/Restore, Pilot Light, Warm Standby, Hot Site
- DMS supports CDC - Continous delivery to constantly upload 
- DataSync - can move data to S3, EFS, FSx Windows/Lustre
- DataSync - can be used to EFS to EFS sync data between regions
- Backup sync data from plenty services to S3
- Backup Vault Lock - same as S3 locks, not possible to delete the backups
- X-Ray helps developers analyze and debug production, distributed applications, such as those built using a microservices architecture.
- Datasync can send data to Glacier!

