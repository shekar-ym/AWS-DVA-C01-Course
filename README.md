# AWS-DVA-C01-Notes
This repo contains my notes that i captured during my preparation for AWS Certified Developer Associate certification exam (DVA-C01). Notes are based on Udemy course from Stephane Maarek https://www.udemy.com/course/aws-certified-developer-associate-dva-c01/
If there is anything wrong, please point them out.
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
   1. Creaetd at container leve. Defines how many tasks should run and how
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
  1. ALB support dynamic port forwarding between container and host - instead of specifying port mappings at each task level.
  2. ALB's dynamic port forwarding - is helps to run multiple same tasks on same EC2 instance.
  
        


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
    
