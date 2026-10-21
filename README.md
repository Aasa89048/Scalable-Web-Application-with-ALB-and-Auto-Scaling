# Scalable Web Application with ALB and Auto Scaling

This project implements a scalable web application on AWS using an Application Load Balancer (ALB), EC2 Auto Scaling, and a custom Amazon Machine Image (AMI). The architecture distributes incoming traffic across application instances and automatically adjusts compute capacity based on demand. It includes public and private subnets, NAT-based outbound connectivity, relational database integration, and AWS Systems Manager for secure instance administration. Additional components, including AWS WAF, CloudWatch monitoring, and SNS notifications, support application security, resource monitoring, and operational alerts.

## Key AWS Services

The list below covers the services used in the project and the intended architecture.

- **VPC:** Public and private subnets, NAT gateways, security groups, and network ACLs (NACLs)
- **EC2 + Auto Scaling Group (ASG):** Launch Template and scaling policies, including target tracking
- **ALB + AWS WAF:** Layer 7 routing and WAF rules intended to help protect against common web application risks, including OWASP Top 10 categories
- **CloudFront:** Intended for caching static assets and reducing latency; implementation still needs confirmation
- **RDS Multi-AZ:** MySQL/PostgreSQL with automated failover; the current evidence shows an RDS MySQL instance, but Multi-AZ failover is unverified
- **Route 53:** Intended alias record pointing to the ALB and health checks; implementation still needs confirmation
- **Systems Manager:** Session Manager for secure instance access without relying on public SSH
- **CloudWatch + SNS:** Monitoring dashboards, alarms, and notifications; delivery should be tested in the live environment

## What I Did

Through this project, I worked on the following AWS concepts and skills:

- Design VPCs with suitable subnet, route table, and NAT gateway configurations
- Build highly available architectures across multiple Availability Zones
- Configure ALB listener rules and target group health checks
- Set up Auto Scaling with target tracking and step scaling policies
- Secure applications with WAF, security groups, and private subnets
- Use Systems Manager Session Manager as a bastion-free alternative for instance access

## Solution architecture

the solution diagram using lucidcharts 
link for the diagram: https://lucid.app/lucidchart/a8ea8d02-caaa-457c-9556-0e68c9b16bd2/edit?viewport_loc=-1542%2C-182%2C2783%2C1558%2Cp1&invitationId=inv_1a8a69d9-b773-4d55-822c-ae71ab1e1f9f

![diagram](Screenshots/lucid-project-diagram.png)


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
| CloudFront and s3 | No screenshot evidence found |
| IRoute 53 and ALB | No IaC files found |

