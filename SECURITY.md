# Security Policy

## Supported Versions

We actively support the following versions with security updates:

| Version | Supported          |
| ------- | ------------------ |
| 1.x.x   | :white_check_mark: |
| < 1.0   | :x:                |

## Security Considerations

### Data Collection

The Last9 Kubernetes Observability Installer collects and forwards the following types of data:

- **Traces**: Application traces and spans (may contain request/response data)
- **Logs**: Container logs from all pods in the cluster
- **Metrics**: Cluster, node, and pod metrics (resource usage, health status)
- **Events**: Kubernetes cluster events

**What is NOT collected:**
- Kubernetes Secrets
- Environment variables (unless explicitly added to instrumentation)
- Sensitive credentials
- Personal identifiable information (unless present in application logs/traces)

### Best Practices

When deploying this stack in production:

1. **Credentials Management**
   - Never commit credentials to version control
   - Use Kubernetes Secrets for sensitive data
   - Rotate tokens and passwords regularly
   - Consider using secret management tools (HashiCorp Vault, AWS Secrets Manager, etc.)

2. **Network Security**
   - All data is transmitted over HTTPS/TLS to Last9
   - Consider using network policies to restrict collector access
   - Review firewall rules for outbound connections

3. **RBAC Permissions**
   - The OpenTelemetry Operator requires cluster-wide permissions
   - The collector requires read access to logs and metrics
   - Review and audit RBAC policies regularly
   - Use least-privilege principle where possible

4. **Log Filtering**
   - Configure the collector to filter sensitive data from logs
   - Use processors to redact PII or credentials
   - See [OpenTelemetry Processor Documentation](https://opentelemetry.io/docs/collector/configuration/#processors)

5. **Namespace Isolation**
   - All components run in the `last9` namespace by default
   - Consider network policies to isolate the namespace
   - Limit access to the `last9` namespace to authorized personnel

### Example: Filtering Sensitive Data

To filter sensitive data from logs, edit the collector configuration:

```yaml
processors:
  attributes:
    actions:
      # Remove sensitive attributes
      - key: password
        action: delete
      - key: api_key
        action: delete
      # Redact sensitive values
      - key: email
        action: hash
```

Apply this configuration:
```bash
kubectl edit cm -n last9 last9-opentelemetry-collector
```

## Reporting a Vulnerability

We take security seriously. If you discover a security vulnerability, please follow these steps:

### DO NOT

- ❌ Open a public GitHub issue
- ❌ Discuss the vulnerability publicly
- ❌ Exploit the vulnerability

### DO

1. ✅ **Email us at**: security@last9.io
2. ✅ **Include**:
   - Description of the vulnerability
   - Steps to reproduce
   - Potential impact
   - Suggested fix (if any)
3. ✅ **Wait for acknowledgment**: We aim to respond within 48 hours

### What to Expect

1. **Acknowledgment**: Within 48 hours of your report
2. **Initial Assessment**: Within 5 business days
3. **Status Updates**: Every 7 days until resolution
4. **Fix Timeline**: Critical issues patched within 30 days

### Disclosure Policy

- We follow responsible disclosure practices
- We will coordinate with you on the disclosure timeline
- Security advisories will be published on GitHub
- We may credit you in the advisory (if you wish)

## Security Updates

Subscribe to security updates:

- 🔔 **Watch this repository** on GitHub
- 📧 **Subscribe to our security mailing list**: security-announce@last9.io
- 📰 **Check our blog**: https://last9.io/blog

## Vulnerability Response Process

1. **Report Received**: Security team is notified
2. **Triage**: Severity assessment (Critical/High/Medium/Low)
3. **Investigation**: Root cause analysis and impact assessment
4. **Fix Development**: Patch created and tested
5. **Release**: Security update published
6. **Disclosure**: Security advisory published (coordinated with reporter)

## Security Features

### Current Security Features

- ✅ TLS/HTTPS for all data in transit
- ✅ RBAC-based access control
- ✅ Namespace isolation
- ✅ No local data persistence (agent mode)
- ✅ Configurable log filtering
- ✅ Support for network policies

### Planned Security Features

- 🔄 mTLS for collector communication
- 🔄 Built-in PII detection and redaction
- 🔄 Audit logging
- 🔄 Integration with secret management tools

## Compliance

This project aims to support compliance with:

- **GDPR**: Data minimization and user privacy
- **SOC 2**: Security controls and monitoring
- **HIPAA**: Healthcare data protection (when configured properly)

**Note**: Compliance is shared responsibility. You must properly configure log filtering and data handling for your specific compliance requirements.

## Third-Party Dependencies

We regularly scan our dependencies for vulnerabilities:

- Dependabot enabled for automated security updates
- Regular dependency audits
- Only use official Helm charts from trusted sources

## Contact

For security-related questions or concerns:

- **Email**: security@last9.io
- **PGP Key**: Available on request
- **Response Time**: 48 hours for security issues

For general questions:
- **Email**: support@last9.io
- **Slack**: [Last9 Community](https://last9.io/slack)

---

Last updated: December 2025
