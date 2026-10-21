# Architecture

**Status:** this is the intended target architecture. The repository presently contains manual web-server/phpMyAdmin setup notes only; it has no infrastructure-as-code or deployment evidence for the AWS services below.

## Design

- A VPC spans two Availability Zones, with public subnets for internet-facing entry/NAT and private subnets for application compute.
- Route 53 provides the application DNS record. CloudFront caches static paths; AWS WAF filters requests before the ALB.
- The ALB terminates HTTPS and routes to healthy EC2 instances in a private-subnet Auto Scaling group.
- The application tier connects to a private Multi-AZ RDS database. The database is never publicly reachable.
- Systems Manager Session Manager provides administrative access without a bastion host.
- CloudWatch collects metrics and evaluates alarms; SNS sends notifications.

See [architecture.mmd](architecture.mmd) for the editable Mermaid source and the README for an embedded view.

## Trust boundaries and traffic

| From | To | Intended traffic |
|---|---|---|
| Internet | CloudFront / ALB entry | HTTPS only |
| ALB security group | App security group | Application port only |
| App security group | DB security group | Database port only |
| Managed EC2 instance | AWS Systems Manager endpoints/services | Required management egress |
| Application/ALB metrics | CloudWatch | Metrics and logs as configured |

Exact ports, CIDRs, endpoint choices, and listener behavior must be set in deployment configuration and reviewed before launch.
