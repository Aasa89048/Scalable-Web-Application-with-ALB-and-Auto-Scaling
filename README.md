# Scalable Web Application with ALB and Auto Scaling

This project implements a scalable web application on AWS using an Application Load Balancer (ALB), EC2 Auto Scaling, and a custom Amazon Machine Image (AMI). The architecture distributes incoming traffic across application instances and automatically adjusts compute capacity based on demand. It includes public and private subnets, NAT-based outbound connectivity, relational database integration, and AWS Systems Manager for secure instance administration. Additional components, including AWS WAF, CloudWatch monitoring, and SNS notifications, support application security, resource monitoring, and operational alerts.

## Key AWS Services

- **VPC:** Public and private subnets, NAT gateways, security groups, and network ACLs (NACLs)
- **EC2 + Auto Scaling Group (ASG):** Launch Template and scaling policies, including target tracking
- **ALB + AWS WAF:** Layer 7 routing and WAF rules intended to help protect against common web application risks
- **CloudFront + S3:** Private S3 origin and Origin Access Control (OAC)
- **RDS:** MySQL database and DB subnet group; Multi-AZ failover is not confirmed by available evidence
- **Route 53:** Hosted zone and alias-record verification evidence
- **Systems Manager:** Session Manager for secure instance access without relying on public SSH
- **CloudWatch + SNS:** Alarms and notification topics; end-to-end notification delivery should be tested

## What I Did

- Designed VPC networking with public/private subnets, route tables, and NAT gateways
- Built an architecture spanning multiple Availability Zones
- Configured an ALB, target groups, health checks, and an Auto Scaling group
- Prepared a custom AMI and Launch Template for application instances
- Worked with WAF, private S3 access, CloudFront OAC, CloudWatch alarms, and SNS topics
- Used Systems Manager Session Manager for instance administration

## Solution architecture

The solution diagram was created using Lucidchart.

[Open the Lucidchart architecture](https://lucid.app/lucidchart/a8ea8d02-caaa-457c-9556-0e68c9b16bd2/edit?viewport_loc=-1542%2C-182%2C2783%2C1558%2Cp1&invitationId=inv_1a8a69d9-b773-4d55-822c-ae71ab1e1f9f)

![Solution architecture](Screenshots/lucid-project-diagram.png)

## Troubleshooting Scenarios

These are common issues a cloud engineer may need to investigate in an AWS environment. They are included as practical runbook examples, not claims that every issue happened in this project.

- **Auto Scaling does not add instances during a load test.**
  1. Check the ASG Activity history for launch errors and confirm the policy uses the metric being tested.
  2. Check the evaluation period, capacity limits, subnet IP availability, and Launch Template settings, then test again.

  [Auto Scaling group](Screenshots/ASG.png) · [Launch Template](Screenshots/php-app-LT.png)

- **S3/CloudFront returns “Access Denied” (403).**
  1. Check the object key and origin, then confirm whether direct S3 access is expected to be blocked because the bucket is private.
  2. Verify OAC and the bucket policy allow the intended CloudFront distribution to read objects, then test the CloudFront URL again.

  [Private S3 bucket](Screenshots/s3raminaprivate.png) · [OAC settings](Screenshots/control-setting-for-s3andcloudfront.png) · [CloudFront working](Screenshots/cloudfrontisworking.png)

- **ALB targets are unhealthy or the app returns 5xx errors.**
  1. Check the target group's health status and reason, and make sure the app is running on the expected port and health-check path.
  2. Verify the instance security group allows traffic from the ALB security group, review application logs, and test through the ALB again.

  [ALB](Screenshots/ALB.png) · [Security groups](Screenshots/Security-groups.png) · [App through ALB](Screenshots/php-app-lb-working%282%29.png)

- **An instance in a private subnet cannot reach an external service.**
  1. Check that the subnet route table points to the correct NAT gateway and that the NAT gateway's public subnet has a route to the Internet Gateway.
  2. If it still fails, check security-group egress, network ACLs, and DNS resolution.

  [VPC](Screenshots/Productionvpc.png) · [NAT gateways](Screenshots/two-natgtws-one-in-each-az.png) · [Private route](Screenshots/route-to-natgtw-from-php-app-private-subnet.png)

My approach would be to check the error and resource status first, narrow down the likely cause, make one careful change at a time, and verify the result.

## Implementation status

| Component | Evidence in repository |
|---|---|
| VPC and subnet layout | [VPC screenshot](Screenshots/Productionvpc.png), NAT and route screenshots |
| NAT gateways | [Two NAT gateways screenshot](Screenshots/two-natgtws-one-in-each-az.png) |
| Private-subnet routing | [Private route table screenshot](Screenshots/route-to-natgtw-from-php-app-private-subnet.png) |
| Security groups | [Security groups screenshot](Screenshots/Security-groups.png) |
| PHP app / Apache | [Working app screenshot](Screenshots/LTami-instance-working.png) and [phpMyAdmin screenshot](Screenshots/php-app-lb-working.png) |
| AMI and Launch Template | [AMI](Screenshots/php-app-ami.png), [Launch Template](Screenshots/php-app-LT.png), [instance setup](Screenshots/config-LTami-instance.png) |
| Systems Manager | [SSM role screenshot](Screenshots/php-app-LT-ssm-connection-role.png) |
| Auto Scaling | [ASG screenshot](Screenshots/ASG.png) |
| Load balancing | [ALB screenshot](Screenshots/ALB.png), [application through the load balancer](Screenshots/php-app-lb-working%282%29.png) |
| Database | [RDS subnet group](Screenshots/db-subnet-group.png), [RDS instance](Screenshots/php-mysql-db.png) |
| AWS WAF | [WAF screenshot](Screenshots/waf-ALB.png) |
| CloudWatch and SNS | [Alarms](Screenshots/cloudwatch-alarms.png), [topics](Screenshots/sns-topics.png) |
| CloudFront and S3 | [CloudFront working screenshot](Screenshots/cloudfrontisworking.png), [private S3 bucket](Screenshots/s3raminaprivate.png), and [OAC settings](Screenshots/control-setting-for-s3andcloudfront.png) |
| Route 53 | [Hosted zone](Screenshots/route53-hostedzone.png), [working alias record](Screenshots/proofroute53aliasrecordworks.png) |
| CloudFormation | No template or successful stack screenshot currently present |

## Step-by-step implementation

See [Steps_documentation.md](Steps_documentation.md) for the implementation walkthrough and linked screenshots.
