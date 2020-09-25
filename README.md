# AWS-DVA-C01-Notes
This repo contains my notes that i captured during my preparation for AWS Certified Developer Associate certification exam (DVA-C01). Notes are based on Udemy course from Stephane Maarek https://www.udemy.com/course/aws-certified-developer-associate-dva-c01/
If there is anything wrong, please point them out.


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
    
