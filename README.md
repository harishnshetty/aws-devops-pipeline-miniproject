# aws-devops-pipeline-miniproject

## App Name
Sonarqube integration with AWS Code Pipeline + Github

## Objective
To perform a sonar scan and integrate the result in Sonarcloud with AWS code pipeline and Github.

## Pre-requisite(s)
1. We must have the AWS account with necessary permission.
2. Add/Manage the requires secrets with AWS "Secret Manager" service. In this project Used secret variables are: `sonartoken`, `HOST`, `Organization`, and `Project`
3. Clone the source code from `git@github.com:VikashChoudahry/sonarqube-aws-codebuild.git`
4. AWS Codebuild needs to be setup. While setting this up, we need to authenticate with Github so that it can read the source code
5. AWS Codepipeline needs to be setup.

## Steps to execute
1. Login to AWS Console.
2. Search for "Code Build".
3. Start the code build.

## Challenge(s) faced:
- I was getting the error as - "CodeBuild cannot find the 0.0.0.0/0 destination for the target internet gateway"
"Fix": VPC setup wasn't done correctly. Post fix, everything started working fine.

## Need help?
For any additional help, please contact me from [here](https://www.learnandshare.live/contact) or write to `servikash@gmail.com`. I would be happy to help you!




## Step 1: Take the Github Developer Token from Github

- Settings -> Developer Settings -> Personal Access Token -> Tokens (Classic)
- Note: aws-desecops-cicd


## Step 2: Create the https://sonarqube.io/ account and create a new project

- https://sonarcloud.io/login
-  Go to the my Profile -> Security -> Generate Token
- get the token and save it in the secret manager

## Step 3: Create the VPC Network

- 1 Public
- 1 Private

## Step 4: Create the S3 Bucket and EFS 

- Create the S3 Bucket to Store Artifacts
- zomato-cicd-report
- Create the EFS to Store the dependency-check data
- Zomato-codebuild-efs

## Step 5: Create the CodeBuild

- Create the Github Connection
- Select the ubuntu Runtime
- rolename: devsecops-zomato-jan-2026
- Select your VPC
- Private Subnet
- Default Security Group
- Vaildate the Setting
- Build specifications: buildspec.yml
- privelieged: Enable this flag if you want to build Docker images or want your builds to get elevated privileges
- Create the S3-Bucket to Store Artifacts
- Enable semantic versioning
- NameSpace: Build ID 
- Create the EFS to Store the dependency-check data
- | identifer | ID | Driectory Path | Mount Point
- | zomato  |   fs-78466212 | / | /efs


## Need to Edit in the BuildSpec.yml before you build start

- Replace the Latest URL of 
- Sonarqube Scanner: https://docs.sonarsource.com/sonarqube-server/10.8/analyzing-source-code/scanners/sonarscanner
- Trivy: https://github.com/aquasecurity/trivy
- Dependency-Check: https://github.com/dependency-check/DependencyCheck/releases
- Replace the EFS ID
- update the Sonar Token  in Secret Manager

Secret key      | Secret value
sonartoken      | sfdf4654
HOST            | https://sonarcloud.io/
Organization    | harishnshetty
Project         | sonarcloud-aws-cp


## Update the Roles

### SCA CodeBuild Role
- SecretsManagerReadWrite
- AmazonEFSCSIDriverPolicy
- AmazonEC2ContainerRegistryFullAccess

### DAST CodePipeline Role
- SecretsManagerReadWrite

### CodePipeline
- AmazonSNSFullAccess
- AmazonS3FullAccess



## What Need to be Delete
- Github Token
- Sonarqube Token
- Aws Code Build --> Settings --> connections --> github connections
- Delete the s3 bucket
- Delete the EFS
