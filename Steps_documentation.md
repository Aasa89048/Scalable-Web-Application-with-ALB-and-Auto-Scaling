# How I Built the AWS Web Application

This document describes how I built the AWS web application, with screenshots in the `Screenshots/` folder documenting key parts of the setup. The screenshots reflect the configuration at the time I captured them; they do not necessarily show the current state of the resources.

## Part 1: Setting up the network

### 1. Reviewed the VPC layout

I started by reviewing the VPC and how its subnets and route tables were arranged. This gave me the network foundation for the load balancer, application instances, and database.

![VPC layout](Screenshots/Productionvpc.png)

I used the AWS console to review the subnet CIDRs, Availability Zones, and route table associations.

### 2. Set up NAT gateways

I set up two NAT gateways, one in each Availability Zone, so resources in private subnets could make outbound connections without being directly reachable from the internet.

![NAT gateways in two Availability Zones](Screenshots/two-natgtws-one-in-each-az.png)

I arranged the private subnet routes to use the intended NAT gateway. NAT gateways can continue to incur charges while they exist.

### 3. Checked the private subnet route

I reviewed the route table used by the PHP application subnet to understand how its outbound traffic reached the NAT gateway.

![Private subnet route to NAT](Screenshots/route-to-natgtw-from-php-app-private-subnet.png)

The route table screenshot records the private subnet's routing setup, including the local VPC route and the default route to NAT. I used this to review the intended routing path rather than treating the screenshot as proof of the current configuration.

### 4. Reviewed the security groups

I created and reviewed the security groups used in the project.

![Project security groups](Screenshots/Security-groups.png)

I used security groups to control traffic between the application, load balancer, and database tiers. The screenshot does not make every port or source range legible, so it should not be treated as a complete record of each rule.

## Part 2: Preparing the application instances

### 5. Created the application AMI

I created a custom AMI to use as the starting image for the PHP application instances.

![PHP application AMI](Screenshots/php-app-ami.png)

I used the AMI as the base image for launching application servers. The screenshot records the image at the time I captured it; it does not independently verify every package or setting included in the image.

### 6. Checked the EC2 instance

I reviewed the EC2 instance I used during application setup.

![Application EC2 instance](Screenshots/LT-ami-instance.png)

The screenshot includes a public IPv4 address. It does not establish whether that address belonged to a temporary instance used to prepare the AMI or to a running application instance. For the intended private-subnet design, I aimed to route application traffic through the ALB and restrict direct inbound access. I also reviewed the instance's security groups and IAM role.

### 7. Installed and configured the web server

I configured the instance for the web application, including the web server setup. My project setup used Apache and PHP, with phpMyAdmin for database administration.

![Web server and application setup](Screenshots/config-LTami-instance.png)

For a repeatable deployment, I would move the setup into a reviewed bootstrap script or configuration-management process. I avoided treating user data as a place to store passwords or other secrets.

### 8. Reviewed the Launch Template settings

I reviewed the Launch Template details and version information before using it to launch application servers.

![Launch Template details](Screenshots/LTami-instance-working.png)

The settings I considered included the AMI, instance type, network configuration, security groups, IAM instance profile, storage encryption, and metadata options. The screenshot records the displayed settings but does not prove that every setting was correct.

### 9. Created the Launch Template

I created a Launch Template to store the instance settings that the Auto Scaling group would use when launching application servers.

![PHP application Launch Template](Screenshots/php-app-LT.png)

### 10. Configured Systems Manager access

I configured the role and access setup for using AWS Systems Manager with the application instance.

![Systems Manager role setup](Screenshots/php-app-LT-ssm-connection-role.png)

I used Session Manager as the preferred way to access the managed instance rather than leaving SSH open to the internet. The screenshot records the role setup; it does not by itself confirm that a Session Manager session was successfully opened or that all permissions were least-privilege.

## Part 3: Load balancing and scaling

### 11. Set up the Auto Scaling group

I configured an Auto Scaling group to use the Launch Template to create and manage application instances.

![Auto Scaling group](Screenshots/ASG.png)

I reviewed the minimum, desired, and maximum capacity, the subnets across Availability Zones, the health-check settings, the target group attachment, and the scaling policy. The screenshot shows the group's configuration but does not by itself prove that a scaling event occurred successfully.

### 12. Set up the Application Load Balancer

I configured the Application Load Balancer as the entry point for requests to the application.

![Application Load Balancer](Screenshots/ALB.png)

I reviewed the listeners, certificate where HTTPS was configured, target group, health-check path and port, selected subnets, and security group. I left the end-to-end application test until after the database setup.

## Part 4: Connecting the database

### 13. Set up the DB subnet group

I configured the RDS DB subnet group using subnets in two Availability Zones.

![RDS DB subnet group](Screenshots/db-subnet-group.png)

I reviewed whether the subnet group belonged to the intended VPC and used private subnets across at least two AZs. A subnet group alone does not guarantee that a database is private, so I considered the route tables and database accessibility settings as well.

### 14. Set up the MySQL database

I set up the RDS MySQL database instance used by the project.

![RDS MySQL database](Screenshots/php-mysql-db.png)

I reviewed the database's public accessibility, Multi-AZ configuration, backups, encryption, maintenance settings, and security group. The screenshot documents the instance view, but it does not demonstrate that a Multi-AZ failover test was performed.

### 15. Tested the application through the ALB

After completing the database setup, I tested the application through the load-balancer endpoint.

![PHP application through the load balancer](Screenshots/php-app-lb-working%282%29.png)

I used the endpoint to check that the application responded through the ALB. A more complete load-balancing test would involve sending several requests and confirming that healthy targets were registered. If the page displays an instance identifier, that can help show whether requests reach different instances; a single screenshot alone does not prove traffic distribution or failover.

## Part 5: Security and monitoring

### 16. Set up AWS WAF

I configured an AWS WAF web ACL for the project and captured its setup.

![AWS WAF web ACL](Screenshots/waf-ALB.png)

I reviewed the association with the intended ALB, the enabled rules, and any scope-down settings. The screenshot does not establish that specific attack traffic was blocked; that would require a separate, controlled test.

### 17. Configured CloudWatch alarms

I created CloudWatch alarms to monitor the project.

![CloudWatch alarms](Screenshots/cloudwatch-alarms.png)

I reviewed the metric, threshold, evaluation period, state, and configured action for the alarms. The screenshot records the alarm configuration, but does not by itself prove that a test condition triggered an alarm as expected.

### 18. Set up SNS topics

I configured SNS topics for project notifications.

![SNS topics](Screenshots/sns-topics.png)

I reviewed whether the CloudWatch alarm actions pointed to the intended topic and whether the relevant email subscriptions were confirmed. The presence of a topic does not by itself confirm that a notification was delivered.

## Part 6: Keeping the S3 bucket private and connecting CloudFront

### 19. Created the private S3 bucket

I created an S3 bucket for static content and kept the bucket private rather than making its objects publicly accessible.

![Private S3 bucket](Screenshots/private-s3.png)

I captured the bucket setup as a record of the configuration at that time. The screenshot alone does not confirm the bucket's current settings or policy.

### 20. Configured Origin Access Control (OAC) for CloudFront

I captured the CloudFront origin access setup used with S3. Origin Access Control allows CloudFront to request objects from the bucket without making the bucket itself public.

![S3 and CloudFront origin access control settings](Screenshots/control-setting-for-s3andcloudfront.png)

I reviewed the S3 origin and OAC configuration. For the intended private-origin setup, the bucket policy needs to allow the intended CloudFront distribution to read objects. The screenshot documents the configuration, but does not independently prove that CloudFront delivery worked or that direct S3 object access was blocked.
