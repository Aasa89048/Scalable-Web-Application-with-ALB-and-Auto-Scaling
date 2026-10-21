# Security design

**Status:** controls below are planned design requirements. The current repository does not contain AWS security configuration proving they are enabled.

- Keep application and database instances in private subnets; expose only the intended load-balancer entry point.
- Use security-group references and least privilege: ALB to app port, app tier to database port, and only required management/service egress.
- Use Systems Manager Session Manager instead of a public bastion or open SSH ingress.
- Require HTTPS with a managed certificate; restrict administrative access and protect DNS changes.
- Associate AWS WAF with the public entry point and configure appropriate managed rule groups, rate controls, logging, and exceptions. Review false positives; do not claim OWASP coverage solely from enabling a rule group.
- Keep database credentials in a managed secret store, rotate them, and never commit secrets.
- Enable encryption at rest and in transit where supported; define backup retention and recovery expectations.
- Apply IAM least privilege, instance profiles, resource tags, and CloudTrail/audit logging.
- Review NACLs and security groups together; NACLs are stateless and require return-path rules.
- Store logs securely and avoid logging credentials, tokens, or sensitive user data.

Before deployment, document concrete ports, IAM policies, encryption settings, WAF rules, backup retention, and log retention, then validate them against the actual AWS environment.
