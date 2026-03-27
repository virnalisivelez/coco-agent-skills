---
name: security-audit
description: Audit code for security vulnerabilities and recommend fixes. Use when reviewing code for OWASP top 10 issues, checking for injection attacks, evaluating authentication flows, or hardening APIs. Produces structured vulnerability reports.
license: Apache-2.0
metadata:
  original-author: terminal-skills
  original-repo: TerminalSkills/skills
  vendored-by: good-stories-llc
  version: "1.0"
---

# Security Audit

Audit codebases for security vulnerabilities across the OWASP Top 10 and beyond. Produce structured reports with severity ratings, proof-of-concept attack descriptions, and actionable remediation steps.

## When to Use

- User asks to review code for security issues
- User wants an audit before deploying to production
- User needs to check API endpoints for common vulnerabilities
- User asks about authentication, authorization, or secrets management

## Audit Methodology

### Phase 1: Reconnaissance

Before reading code, identify the attack surface:

1. **Entry points:** API endpoints, form handlers, file uploads, webhooks, WebSocket handlers
2. **Data flows:** Where does user input enter? Where does it get stored, processed, or displayed?
3. **Trust boundaries:** Frontend vs. backend, internal vs. external services, user roles
4. **Sensitive assets:** Authentication tokens, PII, payment data, API keys, database credentials
5. **Dependencies:** Third-party libraries and their known CVEs

### Phase 2: OWASP Top 10 Checklist

Walk through each category systematically:

#### A01: Broken Access Control

Check for:
- Missing authorization checks on endpoints (especially admin routes)
- Insecure Direct Object References (IDOR): can user A access user B's data by changing an ID?
- Missing function-level access control: can a regular user call admin APIs?
- CORS misconfiguration: overly permissive `Access-Control-Allow-Origin`
- Directory traversal: can path parameters escape the intended directory?
- JWT manipulation: is the signature verified? Can the algorithm be changed to "none"?

Remediation pattern:
```python
# BAD: No authorization check
@app.get("/users/{user_id}/data")
def get_user_data(user_id: int):
    return db.query(User).get(user_id)

# GOOD: Verify the requesting user owns this resource
@app.get("/users/{user_id}/data")
def get_user_data(user_id: int, current_user: User = Depends(get_current_user)):
    if current_user.id != user_id and not current_user.is_admin:
        raise HTTPException(403, "Forbidden")
    return db.query(User).get(user_id)
```

#### A02: Cryptographic Failures

Check for:
- Passwords stored in plaintext or with weak hashing (MD5, SHA-1)
- Sensitive data transmitted without TLS
- Hardcoded encryption keys or secrets
- Weak random number generation for tokens (`random` instead of `secrets`)
- Missing encryption at rest for PII or payment data

Remediation:
- Passwords: bcrypt or argon2 with appropriate work factors
- Tokens: `secrets.token_urlsafe(32)` (Python) or `crypto.randomBytes(32)` (Node)
- Secrets: environment variables or a secrets manager, never in code

#### A03: Injection

Check for:
- **SQL injection:** String concatenation in queries instead of parameterized statements
- **NoSQL injection:** Unsanitized objects passed to MongoDB queries
- **Command injection:** User input passed to `os.system()`, `subprocess.run(shell=True)`, or `exec()`
- **LDAP injection:** Unsanitized input in LDAP queries
- **Template injection:** User input rendered directly in server-side templates

SQL injection remediation:
```python
# BAD: String formatting
cursor.execute(f"SELECT * FROM users WHERE email = '{email}'")

# GOOD: Parameterized query
cursor.execute("SELECT * FROM users WHERE email = %s", (email,))
```

Command injection remediation:
```python
# BAD: Shell injection
os.system(f"convert {filename} output.png")

# GOOD: Use subprocess with list args (no shell)
subprocess.run(["convert", filename, "output.png"], check=True)
```

#### A04: Insecure Design

Check for:
- Missing rate limiting on login, registration, password reset
- No account lockout after failed attempts
- Predictable resource IDs (sequential integers) for sensitive resources
- Business logic flaws: can a user skip payment? Apply a coupon twice?
- Missing re-authentication for sensitive actions (password change, email change)

#### A05: Security Misconfiguration

Check for:
- Debug mode enabled in production (`DEBUG=True`, verbose error pages)
- Default credentials still active
- Unnecessary HTTP methods enabled (TRACE, DELETE on read-only endpoints)
- Missing security headers (see Headers section below)
- Stack traces or internal details leaked in error responses
- Unnecessary ports or services exposed

#### A06: Vulnerable Components

Check for:
- Dependencies with known CVEs (run `pip audit`, `npm audit`, `cargo audit`)
- Outdated frameworks or libraries
- Abandoned dependencies (no updates in 2+ years)
- Dependencies pulled from untrusted sources

#### A07: Authentication Failures

Check for:
- Weak password requirements (no minimum length or complexity)
- Missing multi-factor authentication on admin accounts
- Session tokens that do not expire or rotate
- Session fixation: is a new session ID issued after login?
- Credentials sent in URL query parameters (logged in server logs)
- Password reset tokens that are predictable or never expire

#### A08: Data Integrity Failures

Check for:
- Deserialization of untrusted data (pickle, Java serialization, YAML load)
- Missing integrity checks on software updates or plugins
- CI/CD pipeline accessible without authentication
- Unsigned JWTs accepted

#### A09: Logging and Monitoring Failures

Check for:
- Failed login attempts not logged
- Sensitive data (passwords, tokens, PII) written to logs
- No alerting on suspicious activity (brute force, privilege escalation)
- Logs stored without integrity protection (can an attacker modify them?)

#### A10: Server-Side Request Forgery (SSRF)

Check for:
- User-supplied URLs fetched by the server without validation
- Internal network addresses accessible via URL parameters
- Cloud metadata endpoints reachable (169.254.169.254)

Remediation:
```python
# BAD: Fetch any URL the user provides
response = requests.get(user_provided_url)

# GOOD: Allowlist domains and validate
from urllib.parse import urlparse
ALLOWED_DOMAINS = {"api.example.com", "cdn.example.com"}
parsed = urlparse(user_provided_url)
if parsed.hostname not in ALLOWED_DOMAINS:
    raise ValueError("Domain not allowed")
```

### Phase 3: Security Headers

Verify these HTTP response headers are set:

| Header | Recommended Value | Purpose |
|---|---|---|
| `Strict-Transport-Security` | `max-age=31536000; includeSubDomains` | Force HTTPS |
| `Content-Security-Policy` | Restrictive policy | Prevent XSS |
| `X-Content-Type-Options` | `nosniff` | Prevent MIME sniffing |
| `X-Frame-Options` | `DENY` or `SAMEORIGIN` | Prevent clickjacking |
| `Referrer-Policy` | `strict-origin-when-cross-origin` | Limit referrer leaks |
| `Permissions-Policy` | Disable unused features | Limit browser APIs |

### Phase 4: Secrets Scan

Search the codebase for leaked secrets:

- API keys, tokens, passwords in source files
- `.env` files committed to version control
- Hardcoded credentials in configuration files
- Private keys (SSH, TLS) in the repository
- Connection strings with embedded passwords

Patterns to search for:
```
password\s*=\s*["']
api[_-]?key\s*=\s*["']
secret\s*=\s*["']
token\s*=\s*["']
BEGIN (RSA|DSA|EC|OPENSSH) PRIVATE KEY
```

## Output Format

```markdown
## Security Audit Report

**Scope:** [files/endpoints reviewed]
**Date:** [date]
**Risk Rating:** Critical / High / Medium / Low

### Findings

#### [SEV-001] [Severity: Critical/High/Medium/Low] Title

- **Category:** OWASP A0X - Category Name
- **Location:** `file.py:42`
- **Description:** What the vulnerability is and how it can be exploited.
- **Impact:** What an attacker could achieve (data breach, account takeover, etc.)
- **Remediation:** Specific code change or configuration to fix the issue.
- **Proof of Concept:** Example attack payload or steps to reproduce.

### Summary Table

| ID | Severity | Category | Location | Status |
|---|---|---|---|---|
| SEV-001 | Critical | A03 Injection | api.py:42 | Open |
| SEV-002 | High | A01 Access Control | routes.py:15 | Open |

### Recommendations

1. [Prioritized list of actions]
2. [Start with critical, then high, then medium]
3. [Include both quick wins and longer-term improvements]
```

## Severity Ratings

| Severity | Definition |
|---|---|
| **Critical** | Exploitable remotely without authentication. Data breach, RCE, or full system compromise. Fix immediately. |
| **High** | Exploitable with low-privilege access. Significant data exposure or privilege escalation. Fix within days. |
| **Medium** | Requires specific conditions to exploit. Limited impact or defense-in-depth failure. Fix within weeks. |
| **Low** | Informational or best-practice deviation. Minimal direct impact. Fix as part of regular maintenance. |
