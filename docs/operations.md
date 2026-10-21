# Operations and troubleshooting

These are target operational procedures, not verified procedures for a running deployment.

## Monitoring and notifications

Planned alarms should cover ALB unhealthy hosts and elevated 5xx responses, EC2/ASG capacity and CPU, RDS availability/storage/connections, and application health. Route actionable alarms to an SNS topic with a confirmed subscription. Define thresholds, evaluation periods, notification owners, and a test procedure when implementing them.

## Triage guide

| Symptom | Check |
|---|---|
| Site unreachable | DNS resolution, CloudFront origin/behavior, WAF blocks, ALB listener and security group |
| Targets unhealthy | Target group health-check path/port, app service status, instance security group, application logs |
| Slow or failed scaling | ASG activity history, launch template, instance health, scaling policy metric and cooldown |
| App cannot reach database | DB endpoint/port, DB subnet group, app/DB security group references, credential source |
| No alarm notification | Alarm state/history, SNS topic policy, subscription confirmation, delivery status |
| Cannot administer instance | Systems Manager managed-node status, instance role, agent, and network access to SSM |

Use Session Manager for access when configured; avoid opening SSH to the internet. Redact secrets and personal data from logs and screenshots.
