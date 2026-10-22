# How I Built the AWS Web Application

This document summarizes how I built the AWS web application. The screenshots in `Screenshots/` record key parts of the setup.

## Part 1: Setting up the network

### 1. Set up the VPC

I set up the VPC, subnets, and route tables as the network foundation for the application.

![VPC layout](Screenshots/Productionvpc.png)

### 2. Set up NAT gateways

I set up two NAT gateways, one in each Availability Zone, to provide outbound internet access for resources in private subnets.

![NAT gateways in two Availability Zones](Screenshots/two-natgtws-one-in-each-az.png)

### 3. Set up the private subnet route

I configured the PHP application subnet's route table to route outbound traffic through the NAT gateway.

![Private subnet route to NAT](Screenshots/route-to-natgtw-from-php-app-private-subnet.png)

### 4. Set up the security groups

I created security groups to control traffic between the application, load balancer, and database tiers.

![Project security groups](Screenshots/Security-groups.png)

## Part 2: Preparing the application instances

### 5. Create the application AMI

I created a custom AMI as the base image for the PHP application instances.

![PHP application AMI](Screenshots/php-app-ami.png)

### 6. Set up the EC2 instance

I set up the EC2 instance for application preparation, including its network access, security groups, and IAM role.

![Application EC2 instance](Screenshots/LT-ami-instance.png)

### 7. Install and configure the web server

I installed and configured Apache and PHP on the application instance to run the web application.

![Web server and application setup](Screenshots/config-LTami-instance.png)

### 8. Configure the Launch Template settings

I configured the Launch Template settings, including the AMI, instance type, network settings, security groups, IAM profile, storage, and metadata options.

![Launch Template details](Screenshots/LTami-instance-working.png)

### 9. Create the Launch Template

I created a Launch Template to store the instance configuration used by the Auto Scaling group.

![PHP application Launch Template](Screenshots/php-app-LT.png)

### 10. Set up Systems Manager access

I configured the role and access setup for AWS Systems Manager so I could manage the application instance through Session Manager.

![Systems Manager role setup](Screenshots/php-app-LT-ssm-connection-role.png)

## Part 3: Load balancing and scaling

### 11. Set up the Auto Scaling group

I created and configured the Auto Scaling group to launch and manage application instances using the Launch Template across the selected Availability Zones.

![Auto Scaling group](Screenshots/ASG.png)

### 12. Set up the Application Load Balancer

I configured the Application Load Balancer as the entry point for application traffic, including its listeners, target group, health checks, subnets, and security group.

![Application Load Balancer](Screenshots/ALB.png)

## Part 4: Connecting the database

### 13. Set up the DB subnet group

I configured the RDS DB subnet group with subnets in two Availability Zones.

![RDS DB subnet group](Screenshots/db-subnet-group.png)

### 14. Set up the MySQL database

I created the RDS MySQL database and configured its accessibility, availability settings, backups, encryption, maintenance settings, and security group.

![RDS MySQL database](Screenshots/php-mysql-db.png)

### 15. Test the application through the ALB

After setting up the database, I tested the application through the load-balancer endpoint.

![PHP application through the load balancer](Screenshots/php-app-lb-working%282%29.png)

## Part 5: Security and monitoring

### 16. Set up AWS WAF

I created and configured an AWS WAF web ACL for the project and associated it with the ALB.

![AWS WAF web ACL](Screenshots/waf-ALB.png)

### 17. Set up CloudWatch alarms

I created CloudWatch alarms to monitor the application and configured their metrics, thresholds, evaluation periods, and actions.

![CloudWatch alarms](Screenshots/cloudwatch-alarms.png)

### 18. Set up SNS topics

I created SNS topics for project notifications and configured their connection to CloudWatch alarm actions and email subscriptions.

![SNS topics](Screenshots/sns-topics.png)

## Part 6: Keeping the S3 bucket private and connecting CloudFront

### 19. Create the private S3 bucket

I created a private S3 bucket for static content and kept its objects from being publicly accessible.

![Private S3 bucket](Screenshots/private-s3.png)

### 20. Configure Origin Access Control (OAC) for CloudFront

I configured Origin Access Control for the CloudFront S3 origin so CloudFront could access bucket objects without making the bucket public.

![S3 and CloudFront origin access control settings](Screenshots/control-setting-for-s3andcloudfront.png)

## Part 7: Connecting the custom domain with Route 53

### 21. Create a public hosted zone in Route 53

![Route 53 hosted zone and DNS records](Screenshots/route53-hostedzone.png)

### 24. Verify the Route 53 hosted-zone record

![A record in the Route 53 hosted zone](Screenshots/recordinroute53hostedzone.png)

### 25. Test the A Alias record in Route 53

![Route 53 Test record result showing No Error](Screenshots/evidencealiasrecordworks.png)

### Route 53 alias record working

![Route 53 alias record working](Screenshots/proofroute53aliasrecordworks.png)
