# AWS-DVA-C01-Notes
This repo contains my notes that i captured during my preparation for AWS Certified Developer Associate certification exam (DVA-C01). Notes are based on Udemy course from Stephane Maarek https://www.udemy.com/course/aws-certified-developer-associate-dva-c01/
If there is anything wrong, please point them out.

# AWS Fundamentals IAM + EC2
1. One IAM role per application.
2. SGs are firewalls "outside" EC2.
3. SGs are locked to a region/VPC combination.
4. Try to avoid use of Elasctic IPs == reflects poor architectural decisions.
5. User data script is only run once at the instance first start.
6. EC2 User data script runs with root .

EC2 Instance Launch Types:
1. OnDemand -for uninterrupted work loads
2. Reserved instances - Convertible, Scheduled.
3. Spot instances - for workloads resilient to failure.
4. Great combo - Reserved for baseline + OnDemand and Spot for peaks.
5. Dedicated Hosts - Physical dedicated EC2 server, access to sockets and cores, Allocated for 3 year reservation - BYOL, regulatory and compliance req
6. Dedicated instances - Instances are running on hardware thats dedicated to you, may share h/w with other instances in same account. 
   No control over instance placement. 

ENI
1. Logical component in a VPC == virtual network card.
2. Each ENI has one Primary private IPv4, one or more secondary IPv4.
3. Also can have one Elastic IPv4 per private IPv4 AND one public IPv4
4. One or more SGs
5. A MAC address.
6. ENI can be created independently and attach them on the fly to EC2 instances for failover. When that happens private IP moves from one EC2 to other.
7. Bound to specific AZ. 
8. Each ENI of an Ec2 instance can be associated with different SGs

EC2 Extra
1. AMI - locked to region
2. Bustable instance - to handle unexpected traffic and getting the insurance that it will be handled correctly. 

# AWS Fundamentals : ELB + ASG
1. Vertical Scalability: Non distributed systems like DBs - RDS, Elasticache.
2. Horizontal Scalability: Distributed systems. Goes hand in hand with HA.
3. HA can be passive (RDS Multi AZ) or active (Horizontal scaling)
4. LB are servers which forwards the traffic to multiple servers downstream.
5. LBs can seamlessly handle failures of downstream instances using health checks.
6. Provides SSL termination, enforces stickiness with cookies, HA across AZs
7. LB can separate pubic and private traffic.
8 ELB - EC2 LB - managed LB
9. Health check done on a port and a route (/health is common).
10. LB - can in internal or external
11. LB cannot scale instantaneouly - contact AWS for a "Warm up".
12. 4xx - client induced errors, 5xx- app induced errors, LB error 503 == at capacity or not registered target.
13. Monitoring - ELB access logs, Cloudwatch metrics for aggregate statistics (connections count)

Classic LB (v1)
1. Suppors TCP (Layer 4) and HTTP/S (Layer 7).
2. Health checks are TCP / HTTP based.

ALB
1. Layer 7  only LB
2. Load balancing to multiple HTTP app across machines (target groups).
3. Load balancing to multiple applications on same machine (ex: containers)
4. Support for HTTP/2 and website, support redirects from HTTP to HTTPS
5. Routing tables for different target groups - routing based on path in URL, routing based on hostname in URL, routing based on query string, headers.
6. ALB great fit for micro services and container based application, has dynamic port mapping
7. Target groups - EC2 instances (ASG), ECS tasks, Lambda functions, IP addresses (must be private IPs).
8. X-forwarded-For, X-Forwarded-Port, X-forwarded-Proto - application servers should look for these in headers if they have a requirement.

NLB
1. Layer 4 only LB, Forward TCP & UDP traffic to your instaces, can handle millios of requests per second.
2. Latency ~ 100 ms (vs 400 ms for ALB).
3. Has 1 static IP address per AZ and supports assigning Elastic IP (for whitelisting). AWS can assign and EIP or you can choose.
4. There is no SG associated with NLB unlike ALB. The SG associated with the target instance should allow TCP traffic (over port like 80) from anywhere.

LB Stickiness
1. Works for CLB and ALB. "Cookie" is used for stickiness and has an expiration date.
2. Use case: User does not loose session data.
3. Can bring imbalance to load distribution.
4. For CLB - stickiness at LB level. For ALB - stickiness is at Target group level.

Cross zone LB
1. LB will distributes evenly across al registered instances in all AZ.
2. CLB - disabled by default. No charge for inter AZ data
3. ALB - always ON. Cant be disabled. No charges for inter AZ data.
4. NLB - disabled by default. Charged for inter AZ data if enabled.

SSL/TLS
1. Allows traffic between client LB to be encrypted.
2. TLS is newer version.
3. Public SSL is issued by Certificate Authorities and has an expiration date.
4. LB uses X.509 certificates (SSL/TLS server certificate).
5. HTTPS listener - must specify a default cert
6. Clients can use SNI (Server name indication) to specify the host name they reach.
7. SNI solves the problem of loading multiple SSL certs onto one web server  (to serve multiple websites).
8. SNI requires the client to indicate the hostname of the target server in the intial SSL handshake.
9. Server will then find the correct certificate or return default one.
10. SNI works for ALB, NLB and CloudFront.
11. Multiple SSL certs ===> ALB or NLB. Uses SNI
12. CLB supports only one SSL cert. For CLB to support multiple certs, you need to use multiple CLBs

ELB - Connection Draining
1. For CLB == Connection Draining, For Target group == Deregistration Delay (ALB and NLB).
2. Time to complete "in flight requests" while the instance is de-registering or unhealthy. Default 300 secs (1 to 3600 secs). Can be disabled (value = 0)
3. Stops sending new reqs to the instance which is de-registering.

Auto scaling groups
1. Automatically register new instances with LB.
2. Launch Configuration : AMI + Instance type, EC2 user data, EBS Volumes, SGs, SSH Key pair. 
3. Min size, Max size, Initial capacity, Network + subnets info, LB info, Scaling policies.
4. Auto scaling Alarms - possible to scale ASG based on Cloutwatch alarms.
5. Metrics are computed for overall ASG instances (average CPU).
6. Auto scaling new rules - auto scaling rules managed directly by EC2 - 
7. Target average CPU usage, number of requests on ELB per instance, Average Network in, Average Network out.
8. Auto scaling custom metric - example - number of connected users. 
9. Send custom metric from App on EC2 to CloudWatch, create Cloudwatch alarm which will trigger the scaling policy.
10. To update ASG - provide new launch configuration or launch template.
11. IAM roles attached to SG will get assigned to EC2 instances.

Auto Scaling policies
1. Target Tracking scaling. Example: I wanted average ASG CPU to stay around 40%
2. Simple / Step scaling: When a CloudWatch alarm is triggered (>70%), add 2 units, when <30 %, then remove 1
3. Scheduled Actions: Scale based on know usage patterns.
4. Scaling Cooldown: Cool down period helps to ensure that ASG does not launch or terminate instances before previous scaling activity takes effect. 
5. Defaut cooldown period = 300 secs and can be set based on requirement.

Quiz:
1. SNI (Server Name Indication) is a feature allowing you to expose multiple SSL certs if the client supports it. Read more here: https://aws.amazon.com/blogs/aws/new-application-load-balancer-sni/
2. Network Load Balancers expose a public static IP, whereas an Application or Classic Load Balancer exposes a static DNS (URL)

# EC2 Storage - EBS and EFS
1. EBS -- Its a network drive, locked to an AZ ==> take snapshot and move to other AZ, can be detached from an EC2 instance and attached to another one.
2. EBS - has provisioned capacity (GBs,IOPS). Billed for all the provisioned capacity. Capacity can be increased on the fly.
3. GP2, IO1, ST1 (HDD), SC1 (HDD). Only GP2 and IO1 can be used as boot volumes.

EB2 Volume Types:
1. GP2 - Virtual Desktops, Dev and Test env. 1GB to 16 TB, Max IOPS 16000, Small gp2 can busrt upto 3000, 3 IOPS per GB, 5333 GB = max IOPS
2. IO1 - Critical business apps that require more than 16000 IOPS, Large DB workloads, 4G to 16 TB, Max 32000 IOPS, 50 IOPS per GB
3. ST1 - Streaming work loads, Big data, DWH, Log processig, Apache Kafka. 500 GB to 16 TB, Max IOPS 500, Max throughput 500 MiB/s- can burst
4. SC1 - For infrequently accesses data, Cost important use cases, 500 GB to 16 TB, Max IOPS 250, Max throughput 250 MiB/s- can burst

EBS Volume v Instance store
1. Instance store = ephemeral storage, physically attached disk. Better IO performance, Cache use cases, 
2. Data survives reboot, On stop termination, cant resize, Manual backups

EFS
1. Managed NFS
2. EFS works with EC2 instances in multi AZ. Highly scalable
3. Use cases: Content management, web serving, data sharing, wordpress
4. Compatible with Linux bases AMI, POSIX file system
5. Encryption at rest using KMS
6. 2 performance modes (General purpose - default, Max IO (higher latency)
7. Storage Tiers (life cycle management feature) - Standard (Frequently accessed), EFS-IA, 

EBS v EFS
1. EBS - can be attached to 1 EC2 instance at a time, locked to AZ. GP2 - increase disk size increases IOPS, IO1 - IOPs can be increased independently
2. Migrate EBS vol across AZ: Take snapshot, restore snapshot to another AZ.
3. By default - EC2 instance terminated, EBS is deleted. Can be changed. EBS - whole of proivisoned storage is billed.
4. EFS - can be mounted to 100s of EC2, POSIX, Use EFS-IA to save cost, billed only for what you use

# RDS Aurora Elascticache
1. RDS is a managed service.
2. Read replicas, Multi AZ
3. Storage backed by EBS (g2 or i01)
4. RDS Backups - Daily full backup, 7 days retention (can be upto 35 days)
5. DB Snapshots - manually triggered by user, retention period can be indefinite

Read replicas and MultiAZ
1. Read replicas - for performance, within AZ, Cross AZ and Cross region
2. Read replicas - Async replication betweeen Master and read replica ==> reads are eventually consistent.
3. Read replicas - can be promoted to its own DB
4. Read replicas - Network cost ==> if data goes from one AZ to another. Same AZ -- NO cost
5. MultiAZ - for DR + Availability, SYNC replication, automatics app failover. 
6. MultiAZ - not used for scaling.
7. Read replicas can be setup as MultiAZ for DR.
8. IAM Authentication integrated with RDS

RDS Security and Encryption
1. Encryption at rest: Master and read replicas can be encrypted using AES 256 encryption
2. Encrypton has to be set at launch/create time
3. If master is not encrypted, then read replicas CANNOT be encrypted.
4. In flight encryption: SSL certificates to encrypt data to RDS
5. To enforce SSL for Postgre SQL: set rds.force_ssl=1 in AWS RDS Console (parameter groups).
6. To enforce SSL for MySQL: Within the DB: GRANT USAGE ON *.* TO 'mysqluser'@'%' REQUIRE SSL;
7. Snapshot of unencrypted RDS DB = un-encrypted
8. Snapshot of encrypted RDS DB = encrypted
9. Can copy a snapshot into an encrypted one.
10. To encrypt an un-encrypted RDS DB - Create snapshot, copy snapshot and enable encryption, restore DB from encrypted snapshot, migrate app to new DB
11. RDS DBs are usually deployed in private subnet
12. RDS Security groups = same as EC2 SGs
13. Access Management = IAM policies control access/who manage RDS
14. IAM Auth = RDS MySQL and Postgre SQL. Uses IAM Auth token (15 mins lifetime)

Aurora
1. Postgre and MySQL are supported as Aurora DB
2. HA Native, Failover is instantaneous. 
3. 6 copies across 3 AZ. 4/6 for writes, 3/6 for reads.
4. 1 Master + 15 Aurora Read replicas. Supports CRR.
5. Writer endpoint points to Master.
6. Reader endpoint to Read replicas (can be auto scaled).
7. Encryption at rest sing KMS,in flight using SSL.
8. IAM Authentication using IAM token
9. Aurora serverless - good for infrequent, intermittent and unpredictable workloads. Pay per second.
10. Global Aurora - Aurora CRR, Aurora Global Database (1 Primary region -read/write, 5 secondary regions read only with 16 read replicas per secondary region)
11. After a DB is created. you cant change the VPC selection.

ElastiCache
1. In-memory DBs - Redis, Memcached
2. Write scaling using sharding, Read scaling using Read replicas.
3. Multi AZ with failover capability
4. Cache hit, Cache miss (includes write back to cache). Has cache invalidation.
5. User session store.
6. Redis - multi AZ with auto failure, read replicas, Data durability using AOF persistence - even after cache is restarted, data remains in cache. 
7. Memcached - Muti-node for partitioning of data (sharding), Non-persistent, No backup and restore.
8. For Redis - encryption at rest using KMS, in-transit using Redis AUTH


Elasticache strategies
1. Lazyloading/ Cache-aside/Lazy population: Cache hit from Elasticache. If Cache miss, read from RDS aand write it cache.
   Pros: Only requested data is cached, Node failures are not fatal (but increased latency for cache warming)
   Cosn: Cache miss (3 round trips ==> delay, Stale data - data updated in RDS and not in cache
2. Write Through - Add or update cache when DB is updated.
   Pros: Data in cache is never stale, reads are quick. Write penaly v read penalty
   Cons: Missing data until it is added/updated in the DB, Cache churn - a lot of data may never be read.
3. Combine both the strategies for better results / cost.

Cache Evictions and TTL
1. Cache evictions - Delete item explicitly from cache,  Item is evicted because memory is full and not recently used (LRU), set an item TTL.
2. TTL use cases - Leaderboards, Comments , Activity streams



# Route 53
1. Managed DNS.
2. A record - hostname to IPv4
3. AAAA record - hostname to IPv6
4. CNAME - hostname to hostname
5. Alias - hostname to AWS resource
6. Route 53 can use public domain or private domain names (that can be resolved by your instances in VPC)
7. Route 53 TTL - decide based on requirement.

CNAME vs Alias
1. CNAME - hostname to hostname. ONLY FOR NON ROOT DOMAIN (example: something.mydomain.com NOT mydomain.com)
2. Alias - hostname to AWS resource (app.mydomain.com to abcd.amazonaws.com). WORKS FOR BOTH ROOT AND NOT ROOT DOMAIN (mydomain.com
3. Alias - Free, Native Health check

Simple Routing
1. Use when redirect to a single resource is needed.
2. Cannot attach health checks
3. If multiple values are returned, a random one is chosen by the client.

Weighted Routing
1. Control the % of requests that go to specific end point
2. Health checks can be associated.

Latency Routing Policy
1. Redirect users to server that has the least latency close to user.
2. Latency is evaluated in terms of user to designated AWS region.

Route53 Healtchecks
1. X health checks failed ==> unhealthy (default =3)
2. X health checks passed ==> healthy (default =3) 
3. Healthchecks can be integrated with CloudWatch.
4. Can monitor status of other health checks (calculated health check)
5. Can monitor state of CloudWatch alarm.

Failover Routing
1. Health check mandtory.
2. Failover between Primary and Secondary

Geo-Location Routing
1. Different from latency based
2. Routing based on User location

Multivalue Routing
1. Use when routing traffic to multiple resources
2. Healthchecks association is needed.
3. Upto 8 healthy records returned for each multi value query


# VPC
1. VPC is regional resource.
2. Subnets are AZ level resource. Public subnet, Private subnet
3. Route tabels - to define access to internet and between subnets.
4. Internet Gateway - to provide internet connectitiy to instances in VPC
5. Public Subnet will have a route to IGW
6. NAT Gateway (managed), NAT instances - allow instances in private subnet to acces internet while remaining private.
7. NAT Gateway / NAT instance is deployed in public subnet.
8. Private subnet instances have route to NAT Gateway/ NAT instance.

VPC Security
1. NACL - Attached at subnet level. Has ALLOW and DENY rules. Rules only include IP addresses.
2. SGs - firewall that constols to and from an ENI/ an EC2 instance. Rule include IP addresses and other SGs.
3. Flow Logs - VPC Flow logs, Subnet Flow logs, ENI Flow Logs.
4. Troubleshoot issues - subnet to internet, subnet to subnet, internet to subnet
5. Flows logs can be sent to S3 / Cloudwatch Logs 

VPC Peering, End points, VPC, DX
1. VPC Peering - connect VPCs privately using AWS network.
2. VPC Peering - must not have overlapping CIDR, Not transitive. 
3. VPC Endpoints - Allows to connect AWS services using private network instead of traversing through the internet.
4. VPC Endpoint Gateway - S3 and DynamoDB
5. VPC Endpoint Interface - Other services
6. Site to Site VPN - used to connect onprem to AWS. Connection is encrypted, goes over the internet.
7. Direct Connect - Physical connection between onprem and AWS. Connection is private, secure and fast. Goes over private network
8. DX takes time to estabish.
9. Site-to-Site VPN and DX cannot access VPC Endpoints

# Amazon S3 Introduction
1. S3 Object Key = FULL path to the object. s://my_bucket/my_folder/another_folder/my_file.text
2. Key = prefix + object name. Prefix = my_folder/another_folder, object name = my_file.text.
3. Max Object size = 5 TB. If any object > 5 GB, use multi part upload
4. Metadata, Tags, Version ID

S3 Versioning
1. Enabled at bucket level
2. Any file that is not versioned prior to enabling versioning will have version = null.
3. Suspending versioning does not delete previous versions

S3 Encryption
1. SSE-S3 ==> Must set header: "x-amz-server-side-encryption":"AES256"
2. SSE-KMS ==> Must set header: "x-amz-server-side-encryption":"aws:kms", KMS Advantage: user control + audit trail.
3. SSE-C ==> using data keys managed by customer outside AWS, HTTPS must be used.
4. SSE-C ==> Encryption key must be provided in HTTPS header, for every HTTP request made.
5. Client Side Encryption: Amazon S3 Encryption client library can be used, data should be encrypted before sending it to S3.
6. Client Side Encryption: Client responsible for decryption as well
7. Encryption in transit (SSL/TLS) : S3 exposes both HTTP and HTTSPS Endpoint. Recommended HTTPS
8. HTTPS is mandatory for SSE-C

S3 Security and Bucket Policies
1. IAM User Based
2. Resource Based - Bucket policies, Object ACL, Bucket ACL 
3. Bucket settings for Block Public Access (can be set at account level too)
4. VPC Endpoints (for access from instances within VPC without internet)
5. Logging and Audit: S3 Access Logs, CloudTrial 
6. MFA Delete: for Versioned buckets to delete objects
7. Pre-Signed URLs: Short lived. Premium content for logged in users

S3 CORS
1. Origin = scheme (protocol) + host (domain) + port
2. CORS = Cross Origin Resource sharing
3. The requests wont be fulfilled unless the other origin allows for the requests using CORS Headers (ex: Access-Control-Allow-Origin)
4. if a client does a cross-origin request on S3 bucket, we need to enable correct CORS header
5. You can allow for specific origin or for * (all origins)

S3 Consistency Model
1. Read after write consistency for PUTS of new objects. PUT 200==> GET 200
2. Except GET 404==>PUT 200 ==>GET 404 --> eventual consistency
3. Eventual consistency consistency for DELETES and PUTS of existing objects.
4. PUT 200 ==> PUT 200 ==> GET 200 (might be older version)
5. DELETE 200 ==> GET 200
6. NO WAY TO REQUEST "STRONG CONSISTENCY"

# AWS CLI, SDK, IAM Roles and Policies
1. CLI installation troubleshooting: aws:command not found ===> the aws executable not in the PATH environment variable.
2. PATH allows your system to know where to find "aws" executable.
3. AWS Config ==> creates 2 files @.aws -- config ==> which contains the default region where calls will be sent, credentials ===> contains access key and secret access key
4. AWS CLI on EC2 == right way ==> IAM roles attached to EC2 instances.
5. Inline policies - added to role on top of policy/permission which it already has. Which means these policies cannot be added to other roles. 
6. Inline policies - one to one to a role.
7. AWS CLI Dry Runs: --dry-run option
8. AWS CLI STS Decode Errors : API calls results in long error message, use STS command line "decode-authorization-message" to decode the error.
9. aws sts decode-authorization-message --encoded-message 
10. EC2 instance metadata - http://169.254.169.254/latest/meta-data. You can retrieve the IAM role name but you cannot retrieve IAM policy.
11. http://169.254.169.254/latest -- gives dynamic,meta-data,user-data
12. http://169.254.169.254/latest/meta-data/iam/security-credentials/MyRoleName - when a role is attached to an EC2 instance, the EC2 instance gets a temperory 
   access key, secret access key and token - which will be used by EC2 to perform its actions. These credentials are short lived (1 hour).
13. aws configure --profile
14. MFA with CLI - to use MFA, you must create temperory session and must run STS GetSessionToken API call.
15. AWS SDK - use when ou code against AWS services
16. AWS CLI - uses Python SDK (boto3)
17. If no region is specified or configure a default region - us-east-1 will be chosen by default.
18. AWS Limits/Quotas: API Rate Limits, Service Quotas/Service Limits
19. API Rate Limits - For intermittent errors  / ThrottlingException - use exponential backoff, For consistent errors - use API throttling limit increase
20. Service Quotas - increase by opening a ticket, Service Quotas API to request increase.
21. Throttling Exception - Use exponential backoff. Retry mechanism included in SDK API calls, If using API as is - you must implement retry yourself.
22. Credentials Provider Chain - CLI will look for credentials in following order
    1. Command Line options: --region, --output, --profile
    2. Environment Variables - AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY, AWS_SESSION_TOKEN
    3. CLI Credentials file ~/.aws/credentials
    4. CLI Configuration file: aws configure
    5. Container credentials
    6. Instance profile credentils - for EC2 instance profiles
23. Java SDK (ex) will look for credentials in following order
   1. Environment Variables - AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY
   2. Java System properties - aws.accessKeyId and aws.secretKey
   3. default credentials profile file: @ ~/.aws/credentials
   4. ECS container credentials
   5. EC2 instance profile credentials
24. If working within AWS, use IAM role (EC2 instance roles, ECS task roles, Lambda roles)
25. If working outside AWS, use Environment variables or names profiles
26. If you are not using CLI or SDK to make AWS HTTP API calls, then you need to sign them using SigV4.
    Use HTTP Header Option, Query String option (S3 pre-signed URLs)


# Advanced S3 and Athena
1. S3 MFA-Delete - S3 versioning should be enabled. Needed only to delete an object permanently, to disable versioning
2. Only bucket owner (root account) can enable/disable MFA-Delete. Will need root access keys.
3. S3 Default Encryption v Bucket policies -- Bucket policies are evaluated before default encryption (AES-256, SSE-S3).
4. S3 Access Logs: Any request made tp S3, from any account, authorized or denied will be logged to an S3 bucket.
5. CRR and SRR: Asynchronous replication/synching. Must gove proper IAM persmissions to S3
6. CRR Use case: compliance, lower latency accesss, replication across accounts
7. SRR Use case: log aggregation, live replication between prod and test accounts
8. S3 replication == not retroactive. DELETE operations not replicated, no "chaining" of replication. 
9. S3 pre-signed URLs - can be generated using SDK and CLI.  For downloads (CLI), Uploads (SDK). Validity default 3600 secs
10.Users given a pre-signed URL will inherit the persmissions of person who generated URL. 
11. Pre-signed URL Use cases - access to premium content, download access to an ever changing list of users, temporory access to upload file. 
12. Region needs to be explicitly mentioned while generating pre-signed URL using CLI and also configure sigV4.
13. Amazon Glacier - Item = Archive (upto 40 TB), Archives are stored in vaults.
14. Amazon Glacier: Expedited (1 to 5 mins), Standard (3 to 5 hours), Bulk (5 to 12 hours). Minimum 90 days
15. Amazon Glacier Deep Archive: Standard (12 hours), Bulk (48 hours). Minimum 180 days.
16. If replication is enabled on a bucket, then objects cant be on Glacier Tier.
17. S3 Lifecycle Policies: Transition actions, Expiration actions (to delete older version of files, delete incomplete multi part upload)
18. 3500 PUT/COPY/POST/DELETE and 5500 GET/HEAD requests per second per prefix in a bucket.
19. If SSE-KMS is used, GenerateDataKey API is called when you upload and Decrypt KMS API is called for a download action.
20. You cannot request quota increase for KMS. 
21. Use multi part upload for files > 100 MB. 
22. S3 Transfer acceleration (upload only) - works with multi part upload, uses edge locations
23. S3 Byte range fetches - to speed up downloads. Can be used to retrieve only partial data (ex: head of file).
24. S3 Select and Glacier Select : Retrieve less data using SQL by performing server side filtering. 
25. S3 Event notifications: S3:ObjectCreated, S3:ObjectRemoved, S3:ObjectRestore, S3:Replication ... Use case: generate thumbnails of uploaded images
26. If you want to ensure an event notification is generated for every successful  write, enable versioning on your bucket.
27. S3 Event notifications - Target - SNS, SQS and Lambda.
28. AWS Athena - serverless. BI/analytics/reporting - VPC Flow Logs, ELB Logs, CloudTrial Logs
29. S3 Object Lock: Adopt WORM model, Block an object version deletion for a specified amount of time.
30. Glacier Vault Lock: Adopt WORM model, Lock the policy for future edits, used for compliance and data retention.

# CloudFront

  CloudFront Signed URL/Cookies
  1. To distribute paid shared content.
  2. Includes URL expiration, IP ranges from where the content can be accessed, Trusted signers
  3. Shared content (movie, music) - short time, Private content (to an user): can be years
  4. Signed URL - for individual files (one URL per file), Signed Cookies - for multiple files (one signed cookie for many files)
  5. CloudFront signed URL - allow access to path, no matter the origin. can leverage caching feature
  6. S3 signed URL -Issue a request as the person who pre-signed the URL, Uses IAM key of the signing IAM principal, Limited lifetime.
  
  Quiz:
  CF Signed URL is used to distribute paid content through dynamic CF signed URL generation.

# ECS, EKS, Fargate - Docker in AWS

  Docker
  1. Software development platform to deploy apps
  2. Apps are packaged as containers that can run on any OS (Docker daemon)
  3. Docker images are stored in Docker registry
    Public repo : Docker Hub
    Private repo: Amazon ECR
  4. Many containers running on same host ==> shares the host resources.
  5. Dockerfile ==> build ==> Docker image ==> Docker container
  6. Docker images can be stored in Docker Hub or ECR (push or pull)
    
  Docker Containers Managaement
  1. ECS
  2. Fargate: Amazon's Serverless platform
  3. EKS Amazon's managed K8s (open source)
  
  ECS Clusters
  1. Are logical grouping of EC2 instances
  2. EC2 run ECS agents
  3. ECS agents registers EC2 to a ECS cluster
  4. EC2 instances run a special AMI (Amazon ECS-Optimized Amazon Linux AMI) made for ECS
    
    Demo notes
    > ECS instance linked to one EC2 instance
    > 1 vCPU = available CPUs
    
    > ECS Cluster also creates a ASG
    > User data in ASG launch configuration contains the ecs.config file which has details 
    about which ECS cluster the EC2 gets registered to.
    
   ECS Task Definitions
   1. JSON form - defines "how" ECS runs a Docker container
   2. Contains
        Image Name
        Port Binding between Container and Host
        Memory and CPU
        Env variables
        Networking information
   3. Container port and Host port can be different
   4. ECS Task Role - IAM role which allows ECS task to perform API calls to other AWS services.
      If task cannot pull an image from ECR or cannot access S3 - check the Task Role.
   5. ECS Task Execution Role - This is required by tasks to pull container images and publish container logs to CW on your behalf.
   6. Task Memory and CPU - reserver from ECS Memory and CPU.
   7. Container Definition - Docker image,Image tag,Port mapping (between Host port and Container tag)
   
   ECS Service
   1. Creaetd at container level. Defines how many tasks should run and how in a cluster.
   2. Ensures desired number of tasks are running across fleet of EC2 instances.
   3. Can be linked to ELB/NLB/ALB.
   
    Demo notes
    > Service Type:
      REPLICA - run as many tasks you need across EC2 instances
      DAEMON - Run one task per EC2 instance
    > Deployment Type:
      Rolling update
      Blue Green (powered by Code Deploy)
    > Task Placement
  
  ECS Service with Load Balancers
  1. If you dont specify the host port(0) in port mapping, host port assigned will be random. So every task that is launched 
     will have a different host port mapped.
  2. But with this kind of random ports, it will be difficult to route the traffc. 
  2. ALB support dynamic port forwarding between container and host - helps to route the traffic to random host port and balance the load.
  3. ALB's dynamic port forwarding - is helps to run multiple same tasks on same EC2 instance.
  4. In an ECS service, LB can only be set on service creation.
  5. ALB - allws to use dynamic host port mapping (multiple tasks allowed per container instance). Rule based routing and paths supported.
  6. Can't be achieved with Classic LB (requires static host port mappings (only one task allowed per container instance)). Rule based routing and paths not         supported.
  7. SG of ECS Container service should edited to allow traffic (from all ports) from ALB SG.
 
  ECR
  1. Private Docker image repo
  2. Access controlled through IAM. Permission errors ===> Check IAM policy.
  3. AWS CLI v1 login command (may be asked at the exam)
      $(aws ecr get-login --no-include-email --region eu-west-1)
  4. AWS CLI v2 login command (newer, may also be asked at the exam - pipe)
      aws ecr get-login-password --region eu-west-1 | docker login --username AWS -- password-stdin 1234567890.dkr.ecr.eu-west-1.amazonaws.com

  Fargate
  1. Serverless. No EC2 involved.
  2. Create task definitions and AWS will run containers for you.
  3. To scale, just increase number of tasks. 
  4. No need to configure port mapping. Fargate will take care of dynamic routing.
  
  ECS IAM Deep Dive
  1. EC2 Instance profile is attached to EC2 instance which is part of container instance.
  2. EC2 instance profile is used by ECS agent to make API calls to ECS Service (for registering EC2 to ECS).
  3. Also to send logs to CloudWatch logs and pull images from ECR.
  4. ECS Task runs on EC2 instance and needs ECS Task role.
  5. Each task will have specific role for its purpose.
  6. Task role defined at Task definition role.
  
  ECS Task Placement
  1. Helps to define task placement strategy and task placement contraints.
  2. ECS Service must determine where to place tasks based on CPU, memory and available port. Also to determine which task to terminate during scaling.
  3. For Fargate, AWS takes care of this.
  4. Process to select container instances for task placement
      Instances that satisfy CPU, memory and port requirements
      Instances that satisfy task placement contraints 
      Instances that satisfy task placement strategies
      Select the instance.
  5. Task placement Strategies - 
  Binpack ==> based on amount of CPU/Memory available ==> Cost savings
  Random ==> place the tasks randomly
  Spread ===>  If the specified value is AZ, then tasks are placed evenly on EC2 instances across AZs
  6. Task placement strategies can be mixed together.
  7. Task placement constraints -
  distinctInstance - places task on distinct container instances (One Task per Host)
  memberOf - places task on instances that satisfies an expression (cluster query language)
  
  ECS Autoscaling
  1. Service Auto Scaling: CPU and RAM tracked in CloudWatch at ECS Service level
  2. Target Scaling : scaling based on average CloudWatch metrics
  3. Step scaling: Based on CloudWatch alarms
  4. Scheduled scaling: based on predictable changes
  5. ECS Service scaling is NOT equal to EC2 Auto scaling (instance level)
  6. If you want to do ECS service scaling and EC2 Auto scaling at the same time - ECS Cluster Capacity Provider
  
  ECS Summary and Exam Tips
  1. ECS - used to run Docker containers - ECS Classic, Fargate, EKS
  2. ECS Classic - Must connfigure the file /etc/ecs/ecs.config with the cluster name
  3. ECS Classic - EC2 instance must run ECS agent
  4. ECS Classic - ALB for dynamic port mapping, EC2 instance SG must allow traffic from ALM SG on all ports
  5. ECS tasks - can have IAM roles to execute actions against AWS
  6. SG Operatae at instance level not task level
  7. ECR - integrates with IAM, AWS CLI v1 login command, AWS CLI v2 login command, Docker push and pull
  8. If EC2 instance cannot pull a Docker image, check IAM
  9. Fargate AWS provisons containers for us and assigns them ENI
  10.Fargate tasks can have IAM roles to execute actions against AWS
  11. ECS integrates with CloudWatch logs, logging to be setup at task definition level
  12. Each container will have a different log stream, EC2 instance profile needs to have correct IAM permissions
  13. Use IAM Task Roles for your tasks
  14. Task placement strategies: binpack, random, spread
  15. Service Auto scaling: Target tracking, Step, Scheduled
  16. Cluster Auto scaling through Capacity providers.
  
  ECS Quiz
  Which ECS Config must you enable in /etc/ecs/ecs.config to allow your ECS tasks to endorse IAM roles = ECS_ENABLE_TASK_IAM_ROLE
  
# AWS Elasctic Beanstalk
  1. We still have full control over the configuration.
  2. Managed service. Instance configuration and OS is handled by Beanstalk
  3. Deployment strategy is configurable but performed by Beanstalk.
  4. Developer is responsible for application code only
  5. Application, Application version, Environment name (dev,test,prod)
  6. Supports single and multi docker containers along with other plaforms and languages
  7. RDS  can be created as part of Beanstalk environment or application. But when Beastalk is deleted DB gets deleted. So it is upto you to create an RDS DB ouside Beanstalk. 
  8. As part of Beanstalk, a LB can be chosen only while creating Beanstalk. Not after that.
  
  Deployment Modes
  1. All at once: Fastest, has downtime, suits for development environment, no additional cost
  2. Rolling: Application running below capacity, both version runs simultaneously at some point, no additonal cost, long deployment
  3. Rolling with additonal batches: Application runs at capacity, both version runs simultaneously at some point, Small additional cost, sutis for prod
  4. Immutable: Zero downtime, New version of code deployed to temp ASG, Quick roll back in case of failures, High cost, great for prod. 
  5. Blue/Green: Zero downtime, release facility, Create new "stage" environment and deploy v2, "stage" environment to be validated and roll back if needed
  6. Blue/Green: Route 53 with weighter policies to redirect traffic to stage environment, swap URLs when testng is done.
  7. Deployment process: Describe dependencies and zip it along with code.EB will then deploy the zip on each EC2 and resolve dependencies
  8. Beanstalk Life Cycle Policy - to phase out old application versions. Optionally you can retain source bundle in S3 as part of life cycle policy configuration. 
  9. All parameters set in UI, can also be configured with code using files and these files should be under .ebextensions/ directory in root of the source code.
  10. Such files should in YAML or JSON format and should have .config extension. You can add resources such as RDS, Elasticache, DynamoDB etc.
  11. Any resources managed by .ebextensions get deleted if environment is deleted. 
  12. Beanstalk uses CloudFormation under the hood.
  13. Beanstalk Cloning - clones exact environment to another one.
  14. Beanstalk Migration: ELB type cannot be changed once Beanstalk environment is created. Only configuration of ELB can be updated.
  15. Beanstalk Migration: To migrate, create a new env with same configuration except LB, deploy application code to new environment and do CNAME swap.
  15. RDS with Beanstalk: RDS can be provisioned with Beanstalk, which is great for dev/test but not for prod as DB lifecyle is tied to Beanstalk environment lifeccyle.
  16. RDS with Beanstalk: For prod, create RDS separately and provide its connection string in Beanstalk application. 
  17. Elasctic Beanstalk with Single Docker: Does not use ECS. 
  18. Elasctic Beanstalk with Multi Docker Container: helps to run multiple containers per EC2 instance in ELB. Requires config Dockerrun.aws.json (v2) at the root of source code.
  19 . Dockerrun.aws.json is used to generate ECS task definition.
  20. Beanstalk with HTTPS: Load SSL certificate onto LB (from console) or from code: .ebextensions/securelisterner-alb.config
  21. Web server v Worker Environment: if your application performs tasks that are long to complete,offload these tasks to a dedicated worker environment (SQS + EC2). cron.yaml

# AWS CI/CD: CodeCommit, CodePipeline, CodeBuild, CodeDeploy
1. AWS CodeCommit: storing code, AWS CodePipeline: automating pipeline from code to Beanstalk
2. AWS CodeBuild: to build and test the code, AWS CodeDeploy: to deploy code to EC2 fleets (not Beanstalk).
3. CI : Push code to repo + Code build/test (Jenkis CI, AWS CodeBuild)
4. CD: Deploment happens often and quick (Jenkins CD, CodeDeploy)
5. Code: CodeCommit, Build + Test: CodeBuild, Deploy + Provision: Elastic Beanstalk, User managed EC2 fleet (CloudFormation) - then use CodeDeploy
6. CodePipeline: To orchestrate all above steps.
7. CodeCommit: private Git repos, fully managed, highly available, Secured. Integrates with Jenkins/CodeBuild.
8. CodeCommit Security: interactions are using Git, Authentication: SSH Keys, HTTPS, MFA. Authorization: IAM Policies,
9. CodeCommit Security:  Encryption: repos are encrypted using KMS, HTTPS - in transit, 
10. CodeCommit Security: Cross account: dont share ssh keys, Use IAM Roles + AWS STS (AssumeRole API)
11. CodeCommit Notifications: SNS, Lambda, AWS CloudWatch Event rules* 
12. CodeCommit Notifications: SNS, Lambda use cases: Deletion of branches, trigger for pushes that happens in master branch, Trigger AWS Lambda function to perform codebase analysis
13. CodeCommit Notifications: CloudWatch Event rules: Trigger for pull request updates(created/updated. deleted/commented), Commit comment events.
14. CodeCommit supports SSH Keys and HTTPS Git credentials.

CodePipeline
1. Continouse Delivery. CodePipeline needs "IAM Service Role" to perform its actions.
2. Source: Github / CodeCommit / S3. 
3. Build: Codebuild/ Jenkins
4. Load Testing: 3rd party tools
5. Deploy: AWS Code Deploy / Beanstalk / CloudFormation / ECS
6. Made of stages: Each stage = parallel or sequential actions, manual approvals can be defined.
7. Each pipeline stage create "artifacts" = bunch of files stored in S3 and passed to next stage.
8. Troubleshooting:  CodePipeline state changes generates a CloudWatch events which can create SNS notifications.
9. If CodePipeline fails a stage, pipeline stops.
10. If CodePipeline cant perform an action, check "IAM Service Role" has required permissions
11. Each stage in a CodePipeline can have multiple action groups (parallel or sequential).

CodeBuild
1. Managed build service, pay for usage, uses Docker under the hood.
2. Integrates with KMS for encryption of build artifacts, with IAM for build permissions, VPC for network security, CloudTrial for API logging.
3. Source code - GitHub, CodeCommit, CodePipeline, S3.
4. Build instructions defined in code (buildspec.yml file)
5. Output logs to S3 or AWS CloudWatch logs
5. CloudWatch alarms to detect failed builds and trigger notifications. 
6. Builds can be defined within CodePipeline or CodeBuild itself.
7. buildspec.yml must be in root of your code.
8. CodeBuild phases - Install (Dependencies), Pre build, Build, Post buid.
9. CodeBuild containers by default are launched outside VPC.
10. VPC Configuration like VPC ID, Subnet ID, SG IDs to access resources inside VPC.

CodeDeploy
1. To deploy application to many EC2 instances automatically (EC2 instances not managed by Beanstalk).
2. Managed service
3. Each EC2 instance (or On prem machine) must be running a CodeDeploy agent, which polls the AWS CodeDeploy for work to do. 
4. CodeDeploy sends appspec.yml file (@root of the source code) which specifies how to deploy the code.
5. Then application is pulled form GitHub or S3, EC2 will run the deployment instructions.
6. CodeDeploy agent will report of success/failure of deployment on the instance.
7. CodeDeploy does not provision any resources.
8. Deployment type: In-place or Blue/Greem
9. IAM instance profile: need to give EC2 the permissions to pull from S3/Git hub.
10. Service role: Role for CodeDeploy to perform the deploy
11. appesec.yml: 2 sections. File section - how to source and copy from S3/Git Hub to filesystem
12. Hooks: set of instructions to do deploy the new version. The order of Hooks is Application Stop, DownloadBundle, BeforeInstall, AfterInstall, ApplicationStart, ValidateService*
13. appsec.yml + deployment strategy ===> define how to deploy the applicaton, Use hooks to verify the deployment after each deployment phase. 
14. CodeDeploy rollback: You can specify automated rollback - roll back when deployment fails, when alarm thresholds are met or disable rollbacks.
15. If a roll back happens, CodeDeploy redeploys the last known good revision as new deployment

CodeStar
1. is an integrated solution that regroups; GitHub, CodeCommit, CodeBuild, CodeDeploy, CloudFormation, CodePipeline, CloudWatch.
2. Helps to quickly create "CICD-ready" projects for EC2, Lambda, Beanstalk.


# AWS CloudFormation
1. Declarative way of outlining AWS infra, for any resources
2. Infra as Code, code is version controlled.
3. Cost: Each resource is tagged to track the costs.
4. Templates uploaded to S3 and referenced in CloudFormation.
5. To update a template, re-upload a new version. 
6. Components: Resources (Mandatory)
7. Components: Parameters - for dynamic inputs
8. Components: Mappings - static variables 
9. Components: Outputs - Reference to what is created
10. Components: Conditionals: List of conditions to perform resource creation.
11. Components: Metadata. 
12. Template helpers: References, Functions
13. Resources can reference other resources
14. Can i create dynamic amount of resources? NO.
15. Parameters - a way to provide inputs to CloudFomation template. Use them when you want to reuse your templates, when some inputs cannot be determined ahead of time.
16. Fn::Ref   to reference a parameter. Shorthand !Ref
17. Pseudo parameters: AWS provided parameters. AWS::AccountId, AWS::Region ...
18. Mappings - fixed variables within CF templates, Hardcoded. 
19. Use mappings when you know the values in advance, Use parameters when values are use specific.
20. Accessing Mapping values: Fn::FindInMap
21. Outputs: Optional. Can import the values into other stacks (Fn::ImportValue). Use Export block. Use case: Cross stack collaboration.
22. You cant delete the underlying stack untill all the references are deleted too. 
23. Conditions: to control creation of resources based on conditions.
24. Instrinsic Functions: Ref, Fn::GetAtt, Fn::FindInMap, Fn::ImporValue, Fn::Join, Fn::Sub
25. Fn::Ref - To refer Parameters and resources (gets only resource id like EC2 id).
26. Fn::GetAtt - to know the attributes of resources, for ex : AZ of an EC2 machine.
27. Fn::FindInMap -  to return a named value from a specific key
28. Fn::ImportValue - Import values that are exported in other templates.
29. Fn::Join - join values with a delimiter
30. Fn::Sub - to substitute variables from a text. 
31. Stack creation failes: By default, everything rolls back. Option to disable rollback for troubleshooting purpose. ROLLBACK_COMPLETE - only option to delete the stack.
32. Stack update fails:  stack automatically rolls back to previous known working state. UPDATE_ROLLBACK_COMPLETE - update the stack with fixed template. 
33. ChangeSets: to know what changes are happening before it happens.
34. Nested stacks: stacks which are part of other stacks. They allow you to isolate repeated patterns/ common components in separate stacks and call them from other stacks.
35. Cross stacks - helpful when stacks have different lifecycles - Use Export and Fn:ImporValue.
36. Nested stacks - helpful when components must be re-used. It is not shared and is important to higher level stacks only. re-use how to properly configure an ALB.
37. Stacksets: Create, update and delete stacks across multiple accounts and regions with a single operation. Only Admins can create stacksets.

# AWS Monitoring & Audit: CloudWatch, X-Ray and CloudTrial
CloudWatch:
1. Metrics, Logs, Events, Alarms
2. Metric - a variable to monitor (CPU Utilization, NetworkIn,..), grouped by Namespaces, 
3. Dimension - an attribute of a metric (instance id, envrionment ..), you can have 10 dimensions per metric
4. EC2 memory - custom metric. 
5. Metric resolution: Standard - 1 min, High resolution metric - 1 second (more cost) ==> related API parameter StorageResolution.
6. PutMetricData - API call to push metric to CloudWatch.
7. Exponential back off in case of throttle errors
8. CloudWatch Alarms - to trigger notifications for any metric, alarms can go to Auto scaling, EC2 Actions, SNS notifications.
9. Alarm States: OK, ALARM, INSUFFICIENT_DATA


# AWS Integration and Messaging: SQS, SNS and Kinesis
# AWS Serverless: Lambda
# AWS Serverless: DynamoDB
# AWS Serverless: API Gateway
# AWS Serverless SAM: Serverless Application Model

  
# AWS Other Services
  AWS SES - Simple Email services
  1. To send and receive emails.
  2. Integrates with SNS,SQS and Lambda
  3. Integrates with IAM
  
  Summary of DBs (OLTP, OLAP, NOSQL, CACHE)
  1. RDS (PostgreSQL, MySQL,Oracle, MS, Aurora and Aurora serverless) - Provisioned DB
  2. DynamoDB - Serverless, Managed, Key-value
  3. ElasctiCache - Inmemory DB, Redis, Memcached - Cache capability
  4. Redshift - OLAP - Datawarehousing, Analytics
  5. Neptune - Graph Database
  6. DMS - Database Migration Services
  7. DocumentDB - managed MongoDB for AWS
  
  Amazon Certificate Manager (ACM)
  1. To host public and private SSL cert in AWS
  2. ACM loads SSL cert on following integrations
      LB (inlucing the ones created by Elastic Beanstalk)
      CF distributions
      APIs on API Gateway
  3. ALB have SSL termination. HTTPS ==> ALB SSL Termination ==> private HTTP req =====>Less CPU cost in EC2
  4. ACM can provision and maintain the cert on ALB
    
