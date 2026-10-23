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
- Investigated an AWS account-level restriction blocking CloudFront distribution creation and opened an AWS Support case

## Solution architecture

The solution diagram was created using Lucidchart.

[Open the Lucidchart architecture](https://lucid.app/lucidchart/a8ea8d02-caaa-457c-9556-0e68c9b16bd2/edit?viewport_loc=-1542%2C-182%2C2783%2C1558%2Cp1&invitationId=inv_1a8a69d9-b773-4d55-822c-ae71ab1e1f9f)

![Solution architecture](Screenshots/lucid-project-diagram.png)

## Troubleshooting and Operational Problem-Solving

Cloud engineering is not only about deploying resources. It requires isolating failures, checking evidence, making controlled changes, and verifying recovery. This section distinguishes **an issue encountered during this project** from **runbook scenarios that demonstrate a production troubleshooting approach**. Unverified fixes are not presented as completed work.

### 1. CloudFront distribution creation blocked by AWS account verification

**Status:** Encountered; AWS Support case submitted; final resolution not evidenced.

**Symptom:** Creating a CloudFront distribution was blocked by an account-verification requirement, despite the intended design using a private S3 origin and Origin Access Control.

**Investigation and response:**
1. Distinguished a possible account-level restriction from a distribution configuration error instead of repeatedly changing origin settings without evidence.
2. Reviewed the console error and identified that the blocker required account/support review.
3. Opened an AWS Support case titled **“Account verification required to create CloudFront distribution”** on 20 October 2026.
4. Kept the incident unresolved until AWS confirmed the restriction was cleared and the distribution could be created and tested.

**Operational lesson:** Determine whether a failure is caused by resource configuration, IAM authorization, quotas, or account-level controls. Escalate account-level restrictions through the supported channel and track the final outcome.

**Evidence:** [CloudFront distribution/OAC screenshot](Screenshots/cloudfrontdistributionwithoactos3.png) · [S3/CloudFront access-control settings](Screenshots/control-setting-for-s3andcloudfront.png)

> The support case is a confirmed troubleshooting action. The available evidence does not establish that the account restriction was resolved.

### 2. S3 or CloudFront returns Access Denied (403)

**Status:** Diagnostic runbook; no dedicated Access Denied screenshot or confirmed resolution is currently present in the repository.

**Potential causes:** Missing object or case-sensitive key, incorrect S3 origin, bucket policy that does not authorize the intended CloudFront distribution, OAC mismatch, or missing IAM/KMS permissions.

**Troubleshooting sequence:**
1. Identify whether the 403 occurs on a direct S3 URL or a CloudFront URL. A private bucket refusing direct public access can be expected.
2. Confirm that the requested object exists and the key/capitalization is correct.
3. Confirm CloudFront points to the intended S3 origin and OAC is enabled for that origin.
4. Review the bucket policy for least-privilege s3:GetObject access from the intended CloudFront distribution. Keep S3 Block Public Access enabled when public access is not required.
5. If objects use SSE-KMS, verify the KMS key policy and relevant permissions.
6. Check the default root object, such as index.html, when the distribution root URL should open a page.
7. Re-test through CloudFront and record the error and successful result.

**Evidence:** [S3 bucket](Screenshots/s3bucket.png) · [Private S3 bucket](Screenshots/s3raminaprivate.png) · [OAC settings](Screenshots/control-setting-for-s3andcloudfront.png) · [CloudFront working screenshot](Screenshots/cloudfrontisworking.png)

### 3. ALB target health checks fail or the application returns 5xx errors

**Status:** Production-style runbook; no specific failed-health-check incident is claimed.

**Troubleshooting sequence:**
1. Open the target group and inspect each target's health status and reason code.
2. Verify the application service is running and listening on the configured target and health-check port.
3. Check whether the health-check path returns the configured success code; change the path or matcher only if it matches the intended application behavior.
4. Verify the ALB security group permits client traffic and the instance security group permits the target port from the ALB security group.
5. Review application/web-server logs and compare ALB metrics, including target 5xx responses and request counts.
6. Re-test target health and the application through the ALB endpoint after a correction.

**Evidence of the architecture:** [ALB configuration](Screenshots/ALB.png) · [Security groups](Screenshots/Security-groups.png) · [Application through the ALB](Screenshots/php-app-lb-working%282%29.png)

### 4. Auto Scaling does not launch additional instances during a load test

**Status:** Scaling behavior was tested; the cause of the observed lack of scaling activity was not conclusively established in the available evidence.

**Troubleshooting sequence:**
1. Inspect the ASG Activity history for launch failures, cooldowns, or capacity constraints.
2. Confirm the scaling policy is attached to the correct ASG and uses the intended metric, threshold, and evaluation period.
3. Check the metric used by the policy in CloudWatch. A CPU stress test will not trigger a policy based on another metric, and a short test may not satisfy the evaluation window.
4. Verify minimum, desired, and maximum capacity, subnet IP availability, instance quotas, and Launch Template/AMI permissions.
5. Review EC2 launch failures and instance status checks; confirm new instances register with the target group and pass health checks.
6. Repeat a controlled test and save the metric, ASG activity event, and before/after instance counts.

**Evidence:** [Auto Scaling group](Screenshots/ASG.png) · [Launch Template](Screenshots/php-app-LT.png) · [CloudWatch alarms](Screenshots/cloudwatch-alarms.png)

### 5. CloudFormation stack creation or update fails

**Status:** Runbook guidance; no CloudFormation template or successful stack screenshot is currently present in the repository.

**Troubleshooting sequence:**
1. Open the stack Events and identify the first failed resource event and its status reason; later failures may be consequences of the initial error.
2. Validate the template, parameters, resource names, references, and dependencies. Confirm resource IDs belong to the intended VPC and region.
3. Check IAM permissions for CloudFormation and underlying services, service quotas, and resource-specific constraints.
4. If rollback fails, identify the resource preventing rollback and reconcile out-of-band changes before continuing. Avoid deleting resources blindly.
5. Redeploy after the correction, confirm the final stack status, and test the resources themselves.

**Evidence gap:** Add the actual template and a CloudFormation Events/stack-status screenshot after verifying the deployment. A successful stack status alone does not prove end-to-end application health.

### 6. Private subnet instances cannot reach required external services

**Status:** Runbook guidance based on the project's private-subnet and NAT architecture.

**Troubleshooting sequence:**
1. Identify the destination and port, then test DNS resolution and connectivity from the instance.
2. Inspect the subnet's route table and verify the default route points to the correct NAT gateway.
3. Verify the NAT gateway is available in a public subnet whose route table has a route to an Internet Gateway.
4. Check security-group egress, network ACL inbound/outbound rules, DNS settings, and destination availability.
5. Use VPC Flow Logs, if enabled, to help distinguish rejected traffic from routing or application failures.

**Evidence:** [VPC layout](Screenshots/Productionvpc.png) · [Two NAT gateways](Screenshots/two-natgtws-one-in-each-az.png) · [Private-subnet route](Screenshots/route-to-natgtw-from-php-app-private-subnet.png)

### 7. Route 53 alias resolves incorrectly or the application is unreachable through the domain

**Status:** Alias-record working evidence exists; a DNS failure incident is not claimed.

**Troubleshooting sequence:**
1. Verify the record name, record type, alias target, and hosted zone.
2. Confirm the alias targets the intended load balancer and the ALB is active with healthy targets.
3. Use DNS lookup tools to verify the record resolves, accounting for TTL and resolver caching.
4. Test the ALB endpoint separately from the domain. If the ALB works but the domain does not, focus on DNS; if both fail, investigate the application/ALB path.
5. Save the record configuration and successful domain/application test as evidence.

**Evidence:** [Route 53 hosted zone](Screenshots/route53-hostedzone.png) · [Hosted-zone record](Screenshots/recordinroute53hostedzone.png) · [Alias record working](Screenshots/proofroute53aliasrecordworks.png)

### 8. Monitoring alarm does not notify the operator

**Status:** CloudWatch alarms and SNS topics are documented; end-to-end delivery is not proven by the available screenshots.

**Troubleshooting sequence:**
1. Check the alarm state, metric namespace/dimensions, threshold, statistic, and evaluation periods.
2. Confirm the alarm action references the intended SNS topic and the topic subscription is confirmed.
3. Check SNS delivery status and the destination's spam/quarantine or access restrictions.
4. Trigger a safe controlled test where possible, then verify the path from metric to alarm to SNS notification.
5. Record the alarm transition and notification receipt; a visible alarm alone does not prove delivery.

**Evidence:** [CloudWatch alarms](Screenshots/cloudwatch-alarms.png) · [SNS topics](Screenshots/sns-topics.png)

### Troubleshooting method

Across these scenarios, the repeatable operations workflow is:

1. **Observe:** Capture the exact error, timestamp, affected resource, and impact.
2. **Scope:** Determine whether the issue is application-, network-, IAM-, service-, quota-, or account-level.
3. **Inspect evidence:** Use resource status/reason codes, CloudWatch metrics and logs, configuration, and AWS service events.
4. **Change safely:** Make the smallest reversible change that addresses the suspected cause; avoid broad permissions or exposing private resources as a shortcut.
5. **Verify:** Re-test the original failure path and confirm downstream health, not just a green resource status.
6. **Document:** Record root cause, action, validation, and remaining risk; add sanitized screenshots/logs without credentials or sensitive identifiers.

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
