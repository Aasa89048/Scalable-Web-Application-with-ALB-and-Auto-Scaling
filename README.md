# Scalable Web Application with ALB and Auto Scaling

A hands-on AWS project that builds and demonstrates a PHP web application behind an Application Load Balancer, with EC2 Auto Scaling, a VPC split into public/private subnets, a database, and operational/security services.

> **Status based on repository evidence:** screenshots document AWS resources and a working application/phpMyAdmin view: VPC networking, two NAT gateways, private subnet routing, security groups, a PHP AMI/Launch Template, an Auto Scaling group, an ALB, RDS, WAF, CloudWatch alarms, and SNS topics. One EC2 instance screenshot shows a public IPv4 address; it is not clear whether that instance is a temporary setup host or an application node. Confirm the ASG subnet placement before describing compute as private. Screenshots do not prove resources still exist or every configuration is production-ready. See [Implementation status](#implementation-status).

## What this project demonstrates

- Network isolation with public and private subnets and NAT egress.
- Reusable EC2 configuration through a custom AMI and Launch Template.
- Load balancing and Auto Scaling for a PHP application.
- Relational database connectivity from the application tier.
- Administrative access through AWS Systems Manager in the documented instance setup.
- Web application firewall and basic monitoring/notification resources.

## Key AWS Services

The list below covers the services used in the project and the intended architecture. Some items still need live-console verification; CloudFront and Route 53 are not yet evidenced in the repository, and Multi-AZ failover and specific WAF rule behavior have not been demonstrated.

- **VPC:** Public and private subnets, NAT gateways, security groups, and network ACLs (NACLs)
- **EC2 + Auto Scaling Group (ASG):** Launch Template and scaling policies, including target tracking
- **ALB + AWS WAF:** Layer 7 routing and WAF rules intended to help protect against common web application risks, including OWASP Top 10 categories
- **CloudFront:** Intended for caching static assets and reducing latency; implementation still needs confirmation
- **RDS Multi-AZ:** MySQL/PostgreSQL with automated failover; the current evidence shows an RDS MySQL instance, but Multi-AZ failover is unverified
- **Route 53:** Intended alias record pointing to the ALB and health checks; implementation still needs confirmation
- **Systems Manager:** Session Manager for secure instance access without relying on public SSH
- **CloudWatch + SNS:** Monitoring dashboards, alarms, and notifications; delivery should be tested in the live environment

## Learning Outcomes

Through this project, I worked on the following AWS concepts and skills:

- Design VPCs with suitable subnet, route table, and NAT gateway configurations
- Build highly available architectures across multiple Availability Zones
- Configure ALB listener rules and target group health checks
- Set up Auto Scaling with target tracking and step scaling policies
- Secure applications with WAF, security groups, and private subnets
- Use Systems Manager Session Manager as a bastion-free alternative for instance access

## Solution architecture

The diagram below reflects the screenshot evidence and labels areas that still need verification. Editable source: [docs/architecture.mmd](docs/architecture.mmd).

```mermaid
flowchart LR
  User[User browser] --> ALB[Application Load Balancer<br/>evidenced]
  subgraph VPC[AWS VPC]
    subgraph Public[Public subnets across two AZs]
      ALB
      NAT1[NAT Gateway AZ A<br/>evidenced]
      NAT2[NAT Gateway AZ B<br/>evidenced]
    end
    subgraph Private[Private application subnets]
      ASG[EC2 Auto Scaling group<br/>evidenced]
      AppA[PHP / Apache instances<br/>AMI and Launch Template evidenced]
    end
    RDS[(RDS MySQL<br/>instance evidenced<br/>Multi-AZ unverified)]
    ASG --> AppA
    AppA --> RDS
    AppA --> NAT1
    AppA --> NAT2
  end
  WAF[AWS WAF web ACL<br/>evidenced] -. association/configuration to verify .-> ALB
  SSM[Systems Manager role/access setup<br/>screenshot evidence] -. management .-> AppA
  CW[CloudWatch alarms<br/>evidenced] --> SNS[SNS topics<br/>evidenced]
  AppA -. metrics/logs .-> CW
  DNS[Route 53] -. planned / no screenshot evidence .-> User
  CF[CloudFront] -. planned / no screenshot evidence .-> ALB
```

## Request flow

1. A user opens the application endpoint shown in the project evidence.
2. The ALB receives the request and forwards it to a healthy application target.
3. An EC2 instance runs the PHP/Apache application. The instance configuration is based on a custom AMI and Launch Template.
4. The application connects to the RDS MySQL database.
5. Private instances use NAT for outbound access where configured. NAT gateways are shown in both AZs; exact route associations should be confirmed in the live account.
6. AWS WAF, CloudWatch alarm actions, and SNS notification delivery should be reviewed in the AWS console before claiming the full behavior is validated.

## Implementation status

| Component | Evidence in repository | Current status |
|---|---|---|
| VPC and subnet layout | [VPC screenshot](Screenshots/Productionvpc.png), NAT and route screenshots | The VPC setup is shown. I still need to confirm the exact CIDRs and routes in AWS. One instance screenshot has a public IPv4 address, so I also need to check where the application instances are running. |
| NAT gateways | [Two NAT gateways screenshot](Screenshots/two-natgtws-one-in-each-az.png) | The screenshots show two NAT gateways, one in each AZ. Their current state and the routes using them should be checked in the console. |
| Private-subnet routing | [Private route table screenshot](Screenshots/route-to-natgtw-from-php-app-private-subnet.png) | The screenshot shows a route to a NAT gateway. I’d check the full route table in AWS to make sure the other routes are correct too. |
| Security groups | [Security groups screenshot](Screenshots/Security-groups.png) | The security groups are shown, but the screenshot isn’t clear enough to check every rule. I still need to review the allowed ports and sources. |
| PHP app / Apache | [Working app screenshot](Screenshots/LTami-instance-working.png) and [phpMyAdmin screenshot](Screenshots/php-app-lb-working.png) | The screenshots show the application page and phpMyAdmin login page responding when they were captured. |
| AMI and Launch Template | [AMI](Screenshots/php-app-ami.png), [Launch Template](Screenshots/php-app-LT.png), [instance setup](Screenshots/config-LTami-instance.png) | The AMI, Launch Template, and setup steps are documented. I haven’t shown that the setup can be repeated from scratch yet. |
| Systems Manager | [SSM role screenshot](Screenshots/php-app-LT-ssm-connection-role.png) | The role setup is shown. I still need to confirm that I can connect to the instance using Session Manager. |
| Auto Scaling | [ASG screenshot](Screenshots/ASG.png) | The Auto Scaling group is shown, but there’s no test here showing it launch extra instances or replace an unhealthy one. |
| Load balancing | [ALB screenshot](Screenshots/ALB.png), [application through the load balancer](Screenshots/php-app-lb-working%282%29.png) | The ALB and an application response are documented. I haven’t included a test proving requests are spread across multiple targets. |
| Database | [RDS subnet group](Screenshots/db-subnet-group.png), [RDS instance](Screenshots/php-mysql-db.png) | The RDS MySQL instance and subnet group are shown. Multi-AZ failover, backups, encryption, and public-access settings still need to be checked. |
| AWS WAF | [WAF screenshot](Screenshots/waf-ALB.png) | The web ACL is shown. I still need to confirm its ALB association and test the rules to see how they handle requests. |
| CloudWatch and SNS | [Alarms](Screenshots/cloudwatch-alarms.png), [topics](Screenshots/sns-topics.png) | The alarms and SNS topics are shown. I haven’t included a test confirming that an alarm triggers a notification and the message is delivered. |
| CloudFront and Route 53 | No screenshot evidence found | These are part of the planned design, but I haven’t documented their setup in this repository yet. |
| Infrastructure as code | No IaC files found | The repository currently contains screenshots and documentation, not Terraform or another infrastructure-as-code setup. |

For the step-by-step walkthrough, see [Steps documentation](Steps_documentation.md).

## License

No license file is currently present. Reuse and redistribution permissions are not specified.
