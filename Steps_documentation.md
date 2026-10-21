# How I Built the AWS Web Application

This document summarizes how I built the AWS web application. The screenshots in `Screenshots/` record key parts of the setup.

## Part 1: Setting up the network

### 1. Reviewed the VPC layout

I reviewed the VPC, subnets, and route tables to establish the network foundation for the application.

![VPC layout](Screenshots/Productionvpc.png)

### 2. Set up NAT gateways

I set up two NAT gateways, one in each Availability Zone, to provide outbound internet access for resources in private subnets.

![NAT gateways in two Availability Zones](Screenshots/two-natgtws-one-in-each-az.png)

### 3. Checked the private subnet route

I reviewed the PHP application subnet's route table and its route to the NAT gateway.

![Private subnet route to NAT](Screenshots/route-to-natgtw-from-php-app-private-subnet.png)

### 4. Reviewed the security groups

I created and reviewed security groups to control traffic between the application, load balancer, and database tiers.

![Project security groups](Screenshots/Security-groups.png)

## Part 2: Preparing the application instances

### 5. Created the application AMI

I created a custom AMI as the base image for the PHP application instances.

![PHP application AMI](Screenshots/php-app-ami.png)

### 6. Checked the EC2 instance

I reviewed the EC2 instance I used during application setup, including its network access, security groups, and IAM role.

![Application EC2 instance](Screenshots/LT-ami-instance.png)

### 7. Installed and configured the web server

I configured the application instance with Apache and PHP for the web application.

![Web server and application setup](Screenshots/config-LTami-instance.png)

### 8. Reviewed the Launch Template settings

I reviewed the Launch Template version and its AMI, instance type, network settings, security groups, IAM profile, storage, and metadata options.

![Launch Template details](Screenshots/LTami-instance-working.png)

### 9. Created the Launch Template

I created a Launch Template to store the instance configuration used by the Auto Scaling group.

![PHP application Launch Template](Screenshots/php-app-LT.png)

### 10. Configured Systems Manager access

I configured the role and access setup for AWS Systems Manager so I could manage the application instance through Session Manager.

![Systems Manager role setup](Screenshots/php-app-LT-ssm-connection-role.png)

## Part 3: Load balancing and scaling

### 11. Set up the Auto Scaling group

I configured the Auto Scaling group to launch and manage application instances using the Launch Template, across the selected Availability Zones.

![Auto Scaling group](Screenshots/ASG.png)

### 12. Set up the Application Load Balancer

I configured the Application Load Balancer as the entry point for application traffic and reviewed its listeners, target group, health checks, subnets, and security group.

![Application Load Balancer](Screenshots/ALB.png)

## Part 4: Connecting the database

### 13. Set up the DB subnet group

I configured the RDS DB subnet group with subnets in two Availability Zones.

![RDS DB subnet group](Screenshots/db-subnet-group.png)

### 14. Set up the MySQL database

I set up the RDS MySQL database and reviewed its accessibility, availability settings, backups, encryption, maintenance settings, and security group.

![RDS MySQL database](Screenshots/php-mysql-db.png)

### 15. Tested the application through the ALB

After setting up the database, I tested the application through the load-balancer endpoint.

![PHP application through the load balancer](Screenshots/php-app-lb-working%282%29.png)

## Part 5: Security and monitoring

### 16. Set up AWS WAF

I configured an AWS WAF web ACL for the project and reviewed its association with the ALB and its rules.

![AWS WAF web ACL](Screenshots/waf-ALB.png)

### 17. Configured CloudWatch alarms

I created CloudWatch alarms to monitor the application and reviewed their metrics, thresholds, evaluation periods, and actions.

![CloudWatch alarms](Screenshots/cloudwatch-alarms.png)

### 18. Set up SNS topics

I configured SNS topics for project notifications and reviewed their connection to CloudWatch alarm actions and email subscriptions.

![SNS topics](Screenshots/sns-topics.png)

## Part 6: Keeping the S3 bucket private and connecting CloudFront

### 19. Created the private S3 bucket

I created a private S3 bucket for static content rather than making its objects publicly accessible.

![Private S3 bucket](Screenshots/private-s3.png)

### 20. Configured Origin Access Control (OAC) for CloudFront

I configured Origin Access Control for the CloudFront S3 origin so CloudFront could access bucket objects without making the bucket public.

![S3 and CloudFront origin access control settings](Screenshots/control-setting-for-s3andcloudfront.png)
