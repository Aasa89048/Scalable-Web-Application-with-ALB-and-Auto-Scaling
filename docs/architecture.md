# Architecture

## Evidence-based overview

Repository screenshots document a VPC with public/private networking, two NAT gateways, private-subnet routing, security groups, a PHP application AMI and Launch Template, an Auto Scaling group, an ALB, an RDS MySQL database and subnet group, a WAF web ACL, CloudWatch alarms, SNS topics, and application/phpMyAdmin responses. One EC2 instance summary shows a public IPv4 address; its relationship to the Auto Scaling group is unclear, so the app tier's subnet placement is unverified.

The screenshots are point-in-time records. They do not prove current resource state or all settings. In particular, screenshots do not establish CloudFront or Route 53, RDS Multi-AZ failover, scaling policy behavior, confirmed SNS delivery, complete WAF rule coverage, or security-group least privilege.

The diagram source is [architecture.mmd](architecture.mmd); it is embedded in the [README](../README.md#solution-architecture).

## Request path

1. Browser requests the application endpoint.
2. The Application Load Balancer forwards requests to healthy PHP/Apache targets in the Auto Scaling group.
3. The app connects to the RDS MySQL instance.
4. Private instances use NAT gateways for outbound network access where routes are configured.
5. CloudWatch alarms are intended to notify through SNS.
6. WAF is present in the screenshot evidence; verify its association to the ALB and its rules in the live console.

Route 53 and CloudFront are not evidenced and remain planned.

## Component status

| Component | Evidence | Qualification |
|---|---|---|
| VPC, subnet routing, NAT | Production VPC, NAT gateway, and private route screenshots | Verify AZs and route associations live |
| Security groups | Security group list screenshot | Individual rules need review |
| PHP/Apache and AMI | AMI, instance setup, and working page screenshots | Current package/application details unverified |
| Launch Template and ASG | Console screenshots | Scaling policies and behavior unverified |
| ALB | Console and working endpoint screenshots | Health-check state and distribution unverified |
| RDS MySQL | DB subnet group and instance screenshots | Multi-AZ, encryption, backup, and public-access configuration unverified |
| Systems Manager | Instance role screenshot | Managed-node/session success unverified |
| WAF | Web ACL screenshot | Association/rules/testing unverified |
| CloudWatch/SNS | Alarm and topic screenshots | Actions and delivery unverified |
| CloudFront/Route 53 | No screenshots | Planned / not evidenced |

## Intended trust boundaries

- Internet-facing traffic enters through the intended edge/load-balancer path.
- Application instances should reside in private subnets and accept application traffic only from the ALB security group.
- RDS should be private and accept database traffic only from the application tier.
- Instance administration should use Systems Manager instead of public SSH where configured.
- Outbound access from private instances should be limited to required destinations.

These are design intentions; verify the live security groups, routes, IAM roles, and database settings before describing them as enforced.
