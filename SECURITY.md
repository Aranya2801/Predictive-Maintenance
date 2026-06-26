# Security Policy

## Supported Versions

| Version | Supported          |
|---------|--------------------|
| 1.x     | ✅ Active support  |
| < 1.0   | ❌ End of life     |

## Reporting a Vulnerability

**Please do NOT open a public GitHub issue for security vulnerabilities.**

### Responsible Disclosure Process

1. **Email**: security@predictiq.ai
2. **PGP Key**: Available at https://predictiq.ai/.well-known/security.txt
3. **Response time**: We acknowledge within 48 hours and provide a fix timeline within 5 business days.
4. **Credit**: We credit reporters in our CHANGELOG unless anonymity is requested.

### What to Include

- Description of the vulnerability
- Steps to reproduce
- Potential impact assessment
- Any suggested mitigations

### Scope

**In scope:**
- Authentication/authorization bypasses
- SQL injection or data exfiltration
- Remote code execution
- Model poisoning attacks
- Sensor data tampering
- API rate limit bypasses

**Out of scope:**
- Denial of service (we have rate limiting)
- Issues requiring physical access
- Social engineering

## Security Architecture

- JWT tokens with short TTL (60 min access, 30 day refresh)
- bcrypt password hashing (12 rounds)
- SQL injection prevention via SQLAlchemy ORM
- All secrets in environment variables / Vault / AWS Secrets Manager
- TLS 1.3 for all communications
- RBAC with principle of least privilege
- Audit log for all predictions and data access
- Rate limiting on all API endpoints
- Input validation via Pydantic schemas
- Docker containers run as non-root (UID 1001)
- Read-only root filesystem in production containers
- Network policies restrict inter-service communication

## Dependency Management

- Dependabot configured for automated security updates
- `safety check` runs in CI for known CVEs
- Trivy container scanning in build pipeline
- OWASP dependency-check for Java transitive deps
