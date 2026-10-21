# Security review

The screenshots show security groups, an instance role/Systems Manager setup, and a WAF web ACL. They are useful project evidence but are not enough to certify production security. Verify the actual configuration before making stronger claims.

## Review checklist

- **Network:** application and database instances should be in private subnets; inspect route tables, NACLs, and public IP assignment.
- **Security groups:** permit only the intended paths (ALB to app, app to DB, and required egress). Review exact rules; the screenshot text is not sufficiently legible to certify them.
- **Administration:** verify Systems Manager managed-node registration and a successful Session Manager connection. Avoid public SSH.
- **Database:** verify private accessibility, encryption at rest/in transit, backups, secret storage, and Multi-AZ setting/failover separately.
- **WAF:** verify the web ACL association, rule groups, logging, and test outcomes. A visible WAF ACL alone does not prove OWASP Top 10 coverage.
- **IAM:** inspect instance-profile permissions and scope; grant only required actions.
- **Application:** do not expose phpMyAdmin publicly in a production deployment. Protect it with access controls or remove it when no longer needed.
- **Screenshots:** check for credentials, tokens, sensitive account information, and internal endpoints before adding new images.
- **Unverified edge services:** CloudFront and Route 53 are not evidenced in the current screenshots.

Use managed secret storage for database credentials; do not commit secrets or place them in user data.
