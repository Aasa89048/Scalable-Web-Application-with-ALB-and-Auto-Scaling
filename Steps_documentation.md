# Step-by-step project documentation

This guide follows the project evidence currently stored in `Screenshots/`. Every screenshot has a numbered step heading immediately above it, followed by what the image demonstrates and what to verify. Screenshots are historical evidence and may contain AWS console/account details; confirm they are appropriate to keep public.

> **Evidence limits:** screenshots show configuration pages and outcomes at capture time. They cannot prove present-day resource state, exact rule settings where text is too small, successful failover, scaling policy behavior, or notification delivery unless the image directly shows that test.

## Phase 1 — Build the network

### Step 1 — Review the VPC and its resource map

This VPC console view shows the project network and associated subnets/routes. Use it as the starting point for understanding how the environment is divided.

![Step 1 — VPC resource map and subnet layout](Screenshots/Productionvpc.png)

**Verify:** identify the VPC, subnet CIDRs, AZ placement, and route table associations in the live account. Do not infer specific CIDRs from this scaled screenshot.

### Step 2 — Check the two NAT gateways

The screenshot shows two NAT gateways, intended to provide outbound internet access for private resources while retaining one NAT gateway per AZ.

![Step 2 — NAT gateways in the two Availability Zones](Screenshots/two-natgtws-one-in-each-az.png)

**Verify:** both gateways are available and each private route table uses the NAT gateway in its own AZ if that is the intended design. NAT gateways incur ongoing charges.

### Step 3 — Inspect private application subnet routing

This route table screenshot records a private-subnet route to a NAT gateway.

![Step 3 — Private application subnet route to NAT](Screenshots/route-to-natgtw-from-php-app-private-subnet.png)

**Verify:** the local VPC route remains present, the default route points to the expected NAT, and no private application subnet has an unintended direct internet route.

### Step 4 — Review security groups

The console shows the security groups created for the project.

![Step 4 — Project security groups](Screenshots/Security-groups.png)

**Verify:** inspect inbound and outbound rules individually. The screenshot does not provide enough readable detail to certify port ranges or least privilege. Typical intended paths are ALB to app and app to database only.

## Phase 2 — Prepare application instances

### Step 5 — Create or select the application AMI

The AMI console view records the custom PHP application image used as the basis for EC2 instances.

![Step 5 — PHP application AMI](Screenshots/php-app-ami.png)

**Verify:** confirm the AMI is available in the deployment region, its OS/package versions, and that it contains no credentials or temporary files.

### Step 6 — Configure the instance and web server

This terminal/session screenshot shows package installation and Apache service setup on the instance.

![Step 6 — Install and configure the web server](Screenshots/config-LTami-instance.png)

The earlier repository notes mention Apache, PHP, and phpMyAdmin setup. Prefer a reviewed bootstrap script or configuration management for repeatable builds; do not copy passwords into user data.

### Step 7 — Confirm the instance is running

This EC2 detail view shows an application instance and its network placement.

![Step 7 — Application EC2 instance](Screenshots/LT-ami-instance.png)

**Verify:** confirm the instance is in a private application subnet, has the expected security groups and IAM role, and is not assigned a public IP unless explicitly required.

### Step 8 — Review the Launch Template

The Launch Template console view records the template used by the Auto Scaling group.

![Step 8 — PHP application Launch Template](Screenshots/php-app-LT.png)

### Step 9 — Review Launch Template versions and settings

This screenshot shows the Launch Template detail/version view.

![Step 9 — Launch Template details](Screenshots/LTami-instance-working.png)

**Verify:** check AMI, instance type, network interfaces, security groups, IAM instance profile, storage encryption, metadata options, and version selection. The screenshot alone does not certify each setting.

### Step 10 — Check the Systems Manager role setup

The image records the role/configuration associated with Systems Manager access for the PHP application instance.

![Step 10 — Systems Manager instance role](Screenshots/php-app-LT-ssm-connection-role.png)

**Verify:** confirm the instance is registered as a managed node and Session Manager opens successfully without public SSH. Review attached IAM permissions for least privilege.

## Phase 3 — Scale and route traffic

### Step 11 — Review the Auto Scaling group

The Auto Scaling console screenshot shows the project group and its Launch Template association.

![Step 11 — PHP application Auto Scaling group](Screenshots/ASG.png)

**Verify:** check min/desired/max capacity, subnets across AZs, health-check type/grace period, target group attachment, and scaling policies. No scaling exercise is proven by this screenshot.

### Step 12 — Inspect the Application Load Balancer

The ALB console view records the application load balancer and listener configuration.

![Step 12 — Application Load Balancer](Screenshots/ALB.png)

**Verify:** confirm listener protocol and certificate, target group, health-check path and port, AZ/subnet mapping, and security-group rules. Check target health before testing requests.

### Step 13 — Confirm the application page responds

This browser capture shows the basic application response from an EC2-hosted web server.

![Step 13 — Application response](Screenshots/LTami-instance-working.png)

### Step 14 — Confirm the application is reachable through the load balancer

This browser capture shows the PHP application endpoint served through the load-balancer path.

![Step 14 — PHP application through the load balancer](Screenshots/php-app-lb-working(2).png)

**Verify:** make repeated requests and confirm healthy registered targets serve them. If the application displays an instance identifier, use it to demonstrate distribution; the screenshot alone does not prove traffic balancing or failover.

## Phase 4 — Connect the database

### Step 15 — Create the database subnet group

The RDS console shows a DB subnet group with subnets in two Availability Zones.

![Step 15 — RDS DB subnet group](Screenshots/db-subnet-group.png)

**Verify:** confirm private subnets in at least two AZs, expected VPC, and no unintended public route.

### Step 16 — Review the RDS database

The database console view records the project’s MySQL database instance.

![Step 16 — RDS MySQL database](Screenshots/php-mysql-db.png)

**Verify:** check public accessibility, Multi-AZ setting, backups, encryption, maintenance window, and security group. Multi-AZ failover is not demonstrated in this screenshot.

### Step 17 — Confirm phpMyAdmin can reach the database

This phpMyAdmin console screenshot shows database access from the web application environment.

![Step 17 — phpMyAdmin database connection](Screenshots/php-mysql-db.png)

**Verify:** ensure the connection uses the private RDS endpoint and secrets are not embedded in source, screenshots, or user data. The project should eventually replace interactive/manual DB settings with secure parameter storage.

## Phase 5 — Add edge protection and observability

### Step 18 — Review the WAF web ACL

The WAF console view shows a web ACL associated with the project workflow.

![Step 18 — AWS WAF web ACL](Screenshots/waf-ALB.png)

**Verify:** confirm the ACL is associated with the intended ALB, inspect rule groups and scope-down statements, review sampled requests/logs, and perform an authorized test request. A WAF resource alone does not prove OWASP coverage.

### Step 19 — Review CloudWatch alarms

The screenshot shows CloudWatch alarms configured for the project.

![Step 19 — CloudWatch alarms](Screenshots/cloudwatch-alarms.png)

**Verify:** record each metric, threshold, evaluation period, current state, and alarm action. Trigger a safe test condition where practical and confirm state transitions.

### Step 20 — Review SNS notification topics

The SNS screenshot records notification topics for alarm routing.

![Step 20 — SNS topics](Screenshots/sns-topics.png)

**Verify:** confirm the alarm actions target the intended topic and each email/endpoint subscription is confirmed. The screenshot does not demonstrate message delivery.

## Step 21 — Open phpMyAdmin through the project endpoint

This final browser screenshot shows the phpMyAdmin login page at the load-balanced endpoint. Keep credentials out of public documentation and consider whether a public phpMyAdmin endpoint is appropriate for a portfolio demo.

![Step 21 — phpMyAdmin login page](Screenshots/php-app-lb-working.png)

## Components not evidenced by screenshots

The current screenshot set does not establish that CloudFront or Route 53 is configured. It also does not prove multi-AZ RDS failover, Auto Scaling policy behavior, SNS message delivery, or a complete WAF security test. Add evidence and update the README status table only after confirming those components in the live environment.
