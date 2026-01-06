# AWS DevSecOps Project: CI/CD CodePipeline with ECS Deployment, Security Scans & Automation

## For more projects, check out  
[https://harishnshetty.github.io/projects.html](https://harishnshetty.github.io/projects.html)

[![Video Tutorial](https://github.com/harishnshetty/image-data-project/blob/33e4efe7f837ea6cf9e49ab941728beffd418439/aws-codepipeline1.jpg)](https://youtu.be/BgyYqUXuHuk?si=Gi6vkxhnVJQBILkG)

[![Channel Link](https://github.com/harishnshetty/image-data-project/blob/3115a3809f379842f8518b9b4644981936b0c3aa/aws-codepipeline2.png)](https://youtu.be/BgyYqUXuHuk?si=Gi6vkxhnVJQBILkG)



<!-- ## Step 1: Take the Github Developer Token from Github 

- Settings -> Developer Settings -> Personal Access Token -> Tokens (Classic)
- Note: aws-desecops-cicd -->

---

## Step 1: Create the VPC Network

### Network Design

* 2 Public Subnet
* 2 Private Subnet

### Notes

* Public subnets are typically used for internet-facing resources (e.g., Load Balancers, Bastion Hosts).
* Private subnets are used for application and backend resources to improve security and isolation.


---

## Step 2: Create the [https://sonarqube.io/](https://sonarqube.io/) account and create a new project

### Account & Project Setup

* [https://sonarcloud.io/login](https://sonarcloud.io/login)
* Go to the my Profile -> Security -> Generate Token
* get the token and save it in the secret manager

### Notes

* The generated token will be used by the CI/CD pipeline to authenticate with SonarQube.
* Storing the token in a secret manager helps follow security best practices and prevents credential leakage.

---

## Step 3: Create the S3 Bucket and EFS

### Storage Resources

* Create the S3 Bucket to Store Artifacts
* zomato-cicd-report

### Shared File System

* Create the EFS to Store the dependency-check data
* Zomato-codebuild-efs

### Notes

* The S3 bucket is used to store build artifacts, reports, and pipeline outputs.
* The EFS file system allows persistent storage for dependency-check data across CodeBuild runs.


---

## Step 4: Create the CodeBuild

### Source Configuration

* Create the Github Connection

### Environment Configuration

* Select the ubuntu Runtime
* rolename: devsecops-zomato-jan-2026

### Network Configuration

* Select your VPC
* Private Subnet
* Default Security Group

### Validation

* Vaildate the Setting

### Build Configuration

* Build specifications: buildspec.yml
* privelieged: Enable this flag if you want to build Docker images or want your builds to get elevated privileges

### Artifacts Configuration

* Create the S3-Bucket to Store Artifacts
* Enable semantic versioning
* NameSpace: Build ID

### File System Configuration (EFS)

* Create the EFS to Store the dependency-check data

| identifer | ID          | Driectory Path | Mount Point |
| --------- | ----------- | -------------- | ----------- |
| zomato    | fs-78466212 | /              |   /efs      |

### Notes

* CodeBuild runs inside the selected VPC and private subnet for better security.
* EFS is mounted to persist dependency-check data across builds.
* Privileged mode is required when building Docker images inside CodeBuild.


---

## Need to Edit in the BuildSpec.yml before you build start

### Tool Version Updates

* Replace the Latest URL of
* Sonarqube Scanner: [https://docs.sonarsource.com/sonarqube-server/10.8/analyzing-source-code/scanners/sonarscanner](https://docs.sonarsource.com/sonarqube-server/10.8/analyzing-source-code/scanners/sonarscanner)
* Trivy: [https://github.com/aquasecurity/trivy](https://github.com/aquasecurity/trivy)
* Dependency-Check: [https://github.com/dependency-check/DependencyCheck/releases](https://github.com/dependency-check/DependencyCheck/releases)
* S3 Bucket Name

### Configuration Updates

* Replace the EFS ID
* update the Sonar Token  in Secret Manager

### Secret Manager Configuration

| Secret key   | Secret value                                     |
| ------------ | ------------------------------------------------ |
| sonartoken   | sfdf4654                                         |
| HOST         | [https://sonarcloud.io/](https://sonarcloud.io/) |
| Organization | harishnshetty                                    |
| Project      | sonarcloud-aws-cp                                |

### Notes

* Ensure the scanner URLs always point to the latest stable versions.
* The Sonar token must be securely stored and referenced in `buildspec.yml`.
* Updating the EFS ID is mandatory if you are using a different file system across environments.

---

## Need to Edit in the DAST.yml before you build start

### Tool Version Updates
* ZAP_PROXY_URL  [https://github.com/zaproxy/zaproxy/releases/download/v2.17.0/ZAP_2.17.0_Linux.tar.gz] (https://www.zaproxy.org/download/) 
* S3 Bucket Name
* scanning url [https://staging.harishshetty.xyz/](https://staging.harishshetty.xyz/)

### Configuration Updates
* Replace the EFS ID

---

## Update the Roles

### SCA CodeBuild Role

* SecretsManagerReadWrite
* AmazonEFSCSIDriverPolicy
* AmazonEC2ContainerRegistryFullAccess

### DAST CodePipeline Role

* SecretsManagerReadWrite

### CodePipeline

* AmazonSNSFullAccess
* AmazonS3FullAccess
* CodePipeline-CodeBuild- add this --> `["arn:aws:codebuild:*:970378220457:project/DAST-SCAN"]`

### Notes

* Ensure the roles are attached to the correct AWS services (CodeBuild or CodePipeline).
* Least-privilege access is recommended; add only the permissions required for each stage.
* The CodeBuild project ARN must match the correct AWS account and region.

---

## Step 6: Create the ECS TaskDefinition and Cluster

### ECS Task Definition

* staging-zomato
* cpu : .5
* memory: 1GB
* Container - 1
* Container Name: zomato
* Image: zomato
* port: 80

### Create the ECS Cluster and service

* zomato-cluster
* staging-zomato
* Launch type [FARGATE]

### Networking

* Subnet  [2 Private Subnet]
* Create a new security group [Staging SG]
* Public IP [off]

### Load balancing

* Target-Group [staging-zomato-tg]
* Protocol [HTTP]
* Port [80]
* Health check path [/]

### go-to the ALB

* edit the network private subnet to Public Subnet
* add listern 443
* edit http default 80 to redirect to https

### create the ACM and the Route 53 record

* Create the ACM
* Create the Route 53 record

### code pipeline

* connect the github connection app
* build stage --> select the Other build providers
* Environment variables [PIPELINE_EXECUTION_ID] [#{codepipeline.PipelineExecutionId}]
* Staging deploy --> imagedefinitions.json
* update the Roles permissions in CodePipeline-CodeBuild [for DAST]

### Notes

* ECS Fargate is used to avoid managing EC2 instances.
* ALB handles HTTPS termination using ACM certificates.
* `imagedefinitions.json` enables automated image updates during ECS deployment.
* DAST permissions are required for security scanning during the pipeline execution.

---

# What Need to be Delete

### Credentials & Tokens

* Github Token
* Sonarqube Token

### GitHub & CodeBuild Connections

* Aws Code Build --> Settings --> connections --> github connections
* github connections in the codebuild

### Storage Resources

* Delete the s3 bucket
* Delete the EFS

### SonarQube Cleanup

* Delete the SonarQube API KEY and Project

### Load Balancing & Networking

* delete the load balancer and target group
* delete the Nat Gateway - VPC and Elastic IP

### ECS & Container Resources

* delete the ECS Cluster and service and task Definitions
* delete the ECR

### Messaging Services

* delete the SNS Topic
* delete the SNS Subscription

### CI/CD Services

* delete the codepipeline,codebuild,

### Security & Certificates

* delete the acm
* delete the Route 53 record

### Secrets

* secret manager

### Notes

* Perform deletions in the correct order to avoid dependency errors.
* Ensure no active resources are using the NAT Gateway or EFS before deletion.
* Double-check secrets and tokens are no longer referenced by any pipeline or service.

---

