# Deployment and validation

The repository contains screenshots of AWS console configuration and a working application view. It does not contain infrastructure-as-code or a complete repeatable deployment script, so deployment must currently be reproduced from AWS configuration and the [step-by-step screenshot guide](../Steps_documentation.md).

## Reconstruct the environment

Use the screenshot guide as a reference, not as a substitute for live validation:

1. Review VPC, subnet, route table, NAT gateway, and security group configuration.
2. Prepare the PHP/Apache host image and Launch Template.
3. Configure the instance role and verify Systems Manager managed-node access.
4. Attach an Auto Scaling group to the intended subnets and ALB target group.
5. Configure ALB listeners, target group health checks, and application security groups.
6. Provision/configure RDS and its subnet group; keep credentials outside source control.
7. Associate WAF with the intended resource and configure reviewed rules.
8. Create CloudWatch alarms and SNS actions, then confirm end-to-end notification.
9. Add CloudFront and Route 53 only if they are part of the desired deployed design; these are not shown in the current screenshot set.

Record the AWS region, resource names, chosen versions, and sanitized settings. Never publish credentials, account-specific secrets, or private keys.

## Validation checklist

- Confirm private application instances have no unintended public ingress.
- Check NAT gateways and route-table associations in both AZs.
- Confirm target group health and application response through the ALB.
- Exercise Auto Scaling safely and record the scaling policy and result.
- Confirm app-to-RDS connectivity, DB public-access setting, encryption, backups, and Multi-AZ configuration.
- Confirm a Session Manager session works without opening public SSH.
- Review WAF association/rules and perform a safe test request.
- Trigger or simulate an alarm and confirm SNS message delivery.

Do not report any item as tested until the result has been observed and recorded.

## Cleanup

Delete resources in dependency order and confirm removal in the AWS console:

1. Remove test traffic and scale down/delete the Auto Scaling group.
2. Delete ALB listeners, target groups, and the ALB.
3. Delete WAF associations/ACLs when no longer needed.
4. Delete RDS only after deciding whether to keep a final snapshot; retained backups and snapshots can still cost money.
5. Delete NAT gateways and release unused Elastic IPs.
6. Remove unused Launch Templates, AMIs/snapshots, alarms, SNS topics, logs, and security groups.
7. Review the account for any remaining billable resources.

NAT gateways, EC2, ALB, RDS, WAF, storage, logs, and data transfer may incur charges. Use a budget alert and keep learning environments only while needed.
