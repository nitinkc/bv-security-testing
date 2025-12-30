# BitVelocity Security Testing

This module provides security testing tools and configurations for the BitVelocity platform.

## Structure

```
bv-security-testing/
├── zap/                    # OWASP ZAP configurations
├── dependency-check/       # OWASP Dependency Check configs
├── penetration-tests/      # Manual penetration test scenarios
├── security-gates/         # CI/CD security gate scripts
└── reports/                # Security scan reports
```

## Security Testing Layers

### 1. Static Application Security Testing (SAST)
- **Tool**: SonarQube / SpotBugs
- **What**: Analyzes source code for security vulnerabilities
- **When**: Every commit in CI pipeline
- **Checks**: SQL injection, XSS, hardcoded secrets, insecure crypto

### 2. Dependency Vulnerability Scanning
- **Tool**: OWASP Dependency Check, Snyk
- **What**: Scans dependencies for known CVEs
- **When**: Every build, weekly scheduled scan
- **Action**: Fail build on HIGH/CRITICAL CVEs

### 3. Container Security Scanning
- **Tool**: Trivy, Grype
- **What**: Scans container images for vulnerabilities
- **When**: Before image push, in CI pipeline
- **Checks**: Base image vulnerabilities, package CVEs

### 4. Dynamic Application Security Testing (DAST)
- **Tool**: OWASP ZAP
- **What**: Tests running application for vulnerabilities
- **When**: Nightly in staging environment
- **Checks**: SQL injection, XSS, CSRF, broken auth

### 5. Secrets Scanning
- **Tool**: TruffleHog, GitGuardian
- **What**: Detects secrets in code and git history
- **When**: Pre-commit hook, CI pipeline
- **Checks**: API keys, passwords, tokens, certificates

## Quick Start

### Run OWASP ZAP Scan

```bash
cd bv-security-testing/zap
docker run -v $(pwd):/zap/wrk/:rw \
  -t owasp/zap2docker-stable zap-baseline.py \
  -t http://localhost:8080 \
  -c api-scan-config.yaml \
  -r zap-report.html
```

### Run Dependency Check

```bash
cd bv-core-parent
./mvnw org.owasp:dependency-check-maven:check
```

### Run Trivy Container Scan

```bash
trivy image --severity HIGH,CRITICAL bitvelocity/order-service:latest
```

### Run Secrets Scan

```bash
trufflehog git file://. --only-verified
```

## Security Gates in CI/CD

Security gates automatically enforce security standards:

1. **Pre-commit**: Secrets scanning
2. **PR**: SAST, dependency check
3. **Merge to main**: Container scan, contract tests
4. **Deploy to staging**: DAST scan
5. **Before production**: Manual security review

## Vulnerability Management

### Severity Levels

| Severity | Action | Timeline |
|----------|--------|----------|
| CRITICAL | Block deployment, immediate fix | < 24 hours |
| HIGH | Create issue, fix in next sprint | < 1 week |
| MEDIUM | Create issue, prioritize backlog | < 1 month |
| LOW | Document, fix opportunistically | Best effort |

### Suppression

For false positives or accepted risks:

```xml
<!-- dependency-check/suppression.xml -->
<suppress>
   <notes>False positive - not used in production</notes>
   <cve>CVE-2021-12345</cve>
</suppress>
```

## Penetration Testing Scenarios

See `penetration-tests/` for manual test scenarios:

- SQL Injection attempts
- Authentication bypass tests
- Authorization boundary tests
- CSRF token validation
- Rate limiting tests
- Input validation fuzzing

## Security Champions

Each domain should have a security champion responsible for:
- Reviewing security scan results
- Triaging vulnerabilities
- Implementing security best practices
- Staying updated on security advisories

## Learning Resources

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [OWASP ZAP User Guide](https://www.zaproxy.org/docs/)
- [Container Security Best Practices](https://snyk.io/learn/container-security/)
- [Secure Coding Guidelines](https://owasp.org/www-project-secure-coding-practices-quick-reference-guide/)
