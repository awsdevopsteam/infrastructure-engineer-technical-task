# Cloud Infrastructure Engineer
## Take-home technical assessment

## Introduction
The purpose of this assessment is for **import.io** to ascertain the technical suitability of candidates applying for a Cloud Infrastructure Engineer position.

Please note the following:

 - Use any tools that you think are relevant to the tasks. We have added context around the tools currently in use at import.io (indicated on each task with the :arrow_forward: emoji) but if you are inexperienced with these, please use an alternative that you're comfortable with (e.g. Terraform instead of AWS CDK)
 - We do not expect this test to take more than 2 hours

## Process

 1. Clone this repository to your local device
 2. Push it to a *different* publicly-accessible repository (another Github repo or any other cloud-based Git repository host). Do not fork the repository, as other candidates will be able to see your work
 3. Complete the tasks listed below
 4. Edit this `README.md` file to include brief answers to the questions about the assessment listed below
 5. Push your work to your new repository
 6. Send your recruitment contact a link to the new repository

If you have any questions about this process, please speak to your recruitment contact.

## Tasks

### 1. Docker

```Dockerfile
FROM node:16
WORKDIR /app
COPY package.json ./
COPY tsconfig.json ./
COPY src ./src
RUN ls -a
RUN npm install
EXPOSE 3000
CMD ["npm","run","start:dev"]

```


### 2. CI/CD- used groovy jenkins

 ```Jenkinsfile
pipeline {
    agent { label "master" }
    environment {
        ECR_REGISTRY = "444444000111.dkr.ecr.eu-west-2.amazonaws.com/express:latest"
        APP_REPO_NAME= "Reponame"
    }
    stages {
        stage('Build Docker Image') {
            steps {
                sh 'docker build --force-rm -t "$ECR_REGISTRY/$APP_REPO_NAME:latest" .'
                sh 'docker image ls'
            }
        }
        stage('Push Image to ECR Repo') {
            steps {
                sh 'aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin "$ECR_REGISTRY"'
                sh 'docker push "$ECR_REGISTRY/$APP_REPO_NAME:latest"'
            }
        }
    }
       stage('Deploy') {
            steps {
                sh 'kubectl apply -f deployment.yaml,service.yaml'
            }
        }
    }

    post {
        always {
            echo 'Deleting all local images'
            sh 'docker image prune -af'
        }
    }
}
```
 
 
### 3. Infrastructure as Code- Used Cloudformation 

```yaml
AWSTemplateFormatVersion: 2010-09-09
Description: |
  CloudFormation Template for deploying an EC2 instance with Elastic Load Balancer and ASG using RDS database to an imaginary AWS account.
Parameters:
  MyVPC:
    Description: VPC Id of your existing account
    Type: AWS::EC2::VPC::Id

  KeyName:
    Description: write your Key Pair
    Type: AWS::EC2::KeyPair::KeyName

  MySubnets:
    Description: Please select your subnets
    Type: List<AWS::EC2::Subnet::Id>

Resources:
  ALBSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Enable HTTP (80) for Application Load Balancer
      VpcId: !Ref MyVPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0

  WebServerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Enable HTTP for Flask Web Server and SSH for getting into EC2
      VpcId: !Ref MyVPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
       - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 107.22.40.20 
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 18.215.226.36

  WebServerLT:
    Type: AWS::EC2::LaunchTemplate
    Properties:
      LaunchTemplateData:
        ImageId: ami-0c2b8ca1dad447f8a
        InstanceType: t2.micro
        KeyName: !Ref KeyName
        SecurityGroupIds:
          - !GetAtt WebServerSecurityGroup.GroupId
        TagSpecifications:
          - ResourceType: instance
            Tags: 
              - Key: Name
                Value: !Sub Web Server of ${AWS::StackName} Stack
        
  WebServerTG:
    Type: "AWS::ElasticLoadBalancingV2::TargetGroup"
    Properties:
      Port: 80
      Protocol: HTTP
      TargetType: instance
      UnhealthyThresholdCount: 3
      HealthyThresholdCount: 2
      VpcId: !Ref MyVPC
  
  ApplicationLoadBalancer:
    Type: "AWS::ElasticLoadBalancingV2::LoadBalancer"
    Properties:
      IpAddressType: ipv4
      Scheme: internet-facing
      SecurityGroups:
        - !GetAtt ALBSecurityGroup.GroupId
      Subnets: !Ref MySubnets
      Type: application

  ALBListener:
    Type: "AWS::ElasticLoadBalancingV2::Listener"
    Properties:
      DefaultActions: 
        - TargetGroupArn: !Ref WebServerTG
          Type: forward
      LoadBalancerArn: !Ref ApplicationLoadBalancer 
      Port: 80 
      Protocol: HTTP 

  WebServerASG:
    Type: "AWS::AutoScaling::AutoScalingGroup"
    Properties:
      AvailabilityZones:
        !GetAZs ""
      DesiredCapacity: 2
      HealthCheckGracePeriod: 90
      HealthCheckType: ELB
      LaunchTemplate:
        LaunchTemplateId: !Ref WebServerLT
        Version: !GetAtt WebServerLT.LatestVersionNumber
      MaxSize: 3
      MinSize: 1 
      TargetGroupARNs:
        - !Ref WebServerTG

  MyDBSecurityGroup:
    Type: AWS::RDS::DBSecurityGroup
    Properties:
      GroupDescription: This is for Frontend access to RDS instance
      DBSecurityGroupIngress:
        - CIDRIP: 0.0.0.0/0
        - EC2SecurityGroupId: !GetAtt WebServerSecurityGroup.GroupId 

  MyDatabaseServer:
    Type: AWS::RDS::DBInstance
    Properties: 
      AllocatedStorage: 20
      AllowMajorVersionUpgrade: false
      AutoMinorVersionUpgrade: true
      BackupRetentionPeriod: 0
      DBInstanceClass: db.t2.micro
      DBInstanceIdentifier: technical task
      DBName: task-1
      DBSecurityGroups: 
        - !Ref MyDBSecurityGroup
      DeletionProtection: true
      Engine: mysql
      EngineVersion: 8.0.19
      MasterUsername: admin
      MasterUserPassword: osman2745
      Port: 3306
      PubliclyAccessible: true

Outputs:
  WebsiteURL:
    Value: !Sub 
      - http://${ALBAddress}
      - ALBAddress: !GetAtt ApplicationLoadBalancer.DNSName
    Description: TASK Application Load Balancer URL

```



## Questions

 1. How long did you spend on this assessment in total?\
   2 hours

 2. What was the most difficult task?\
 
 First one 

 3. If you had an unlimited amount of time to complete this task, what would you have done differently?\
 I may use terraform

