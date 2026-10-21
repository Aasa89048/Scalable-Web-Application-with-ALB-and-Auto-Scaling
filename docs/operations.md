# Operations and troubleshooting

This guide pairs current screenshot evidence with checks to perform against the live AWS environment. Screenshots may be stale; always check current resource state.

## Monitoring and notifications

CloudWatch alarms and SNS topics are visible in the repository screenshots. Verify each alarm's metric, threshold, period, evaluation count, current state, and action. Confirm subscriptions and test delivery before claiming alerts work end-to-end.

## Triage

| Symptom | Checks |
|---|---|
| Application not reachable | DNS (if configured), WAF association/blocks, ALB listener, security groups, target health |
| Targets unhealthy | Health-check path/port, Apache/PHP service status, instance logs, ALB-to-app rule |
| Auto Scaling not replacing/scaling | ASG activity history, Launch Template version, health checks, desired/min/max, scaling policies |
| Application cannot connect to RDS | RDS status/endpoint, DB subnet group, app and DB security-group rules, credential source |
| No alarm notification | Alarm history/action, SNS topic policy, subscription confirmation and delivery status |
| Session Manager unavailable | Instance profile, managed-node status, SSM agent, network egress or VPC endpoints |

Use Systems Manager for administration if the instance is configured as a managed node. Avoid public SSH where possible. Redact tokens, passwords, connection strings, and personal data from logs and screenshots.
