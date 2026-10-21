# Deployment

**Current state:** there is no reproducible deployment automation in this repository. The commands in the original README are manual host setup notes, not a complete deployment procedure. Do not treat this page as a deployable runbook until infrastructure code and tested commands are added.

## Planned deployment sequence

1. Choose an IaC tool and document versions and AWS prerequisites.
2. Create the VPC, two-AZ subnet layout, routing, NAT egress, and security groups.
3. Create IAM instance roles, Systems Manager access, Launch Template, and Auto Scaling group.
4. Create the ALB, target group, HTTPS listener/certificate, health checks, and scaling policies.
5. Create private Multi-AZ RDS; store connection details in a secrets service rather than source code.
6. Add Route 53, CloudFront, WAF, CloudWatch alarms/dashboard, and SNS subscriptions.
7. Deploy the application and validate each item in the README checklist.
8. Record actual outputs, screenshots or sanitized evidence, and rollback steps.

## Prerequisites to document

- AWS account/region and required permissions
- IaC/runtime versions
- DNS domain ownership and TLS certificate location
- Secure database credential provisioning
- Budget alert and resource naming/tagging convention

Never commit credentials, private keys, account IDs, or unredacted connection strings.

## Cleanup

When deployment automation is added, provide a reviewed destroy command and use it to remove resources in dependency order. Until then, clean up manually and verify the state in the AWS console/CLI:

- Remove test instances and Auto Scaling groups, then Launch Templates if unused.
- Remove ALB/listeners/target groups and CloudFront/WAF resources after detaching associations.
- Remove RDS only after deciding whether snapshots must be retained; snapshots may continue to incur charges.
- Remove NAT gateways, elastic IPs, VPC endpoints, and other billable network resources.
- Remove DNS records only when no longer needed; verify logs, snapshots, and storage separately.

Check the account for remaining billable resources after cleanup.
