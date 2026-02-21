---
name: security
description: >
  Security audit for Sitecore, Umbraco, and Optimizely CMS projects. Scans for OWASP Top 10
  vulnerabilities, CMS-specific security issues, and known dependency CVEs. Use when asked to:
  run a security audit, check for vulnerabilities, scan for security issues, review security,
  or check dependencies for CVEs. Triggers on "security audit", "vulnerability scan",
  "security review", "CVE check", "dependency scan", "OWASP".
user-invocable: true
argument-hint: "[scope: all | code | deps | config]"
allowed-tools: Read, Grep, Glob, Bash
---

# CMS Security Audit

Audit CMS projects for OWASP Top 10 vulnerabilities, CMS-specific security issues, and dependency CVEs.

## Usage

```
/plugin-cms-toolkit:security                # Full audit (code + deps + config)
/plugin-cms-toolkit:security code           # Source code analysis only
/plugin-cms-toolkit:security deps           # Dependency vulnerability scan only
/plugin-cms-toolkit:security config         # Configuration security review only
```

## Workflow

### 1. Detect CMS and Gather Context

Determine the CMS platform, then identify:
- `.csproj` files → NuGet package versions
- `package.json` / `package-lock.json` → npm package versions
- `appsettings*.json`, `web.config` → configuration files
- Source code files (controllers, views, services, API endpoints)

### 2. Scan Source Code

Read the OWASP reference from [owasp-reference.md](owasp-reference.md) and CMS-specific patterns from [cms-security.md](cms-security.md) before scanning.

#### OWASP Top 10 Checks

**A01: Broken Access Control**
- Controllers/endpoints missing `[Authorize]` or equivalent auth attribute
- Direct object references using user-supplied IDs without ownership validation
- CORS misconfiguration (`AllowAny` origin in production)
- Missing anti-forgery tokens on state-changing forms
- API endpoints accessible without authentication

**A02: Cryptographic Failures**
- Hardcoded connection strings, API keys, secrets in source code
- Passwords in plaintext or reversible encryption
- HTTP URLs for sensitive data (check config, API calls)
- Weak hashing algorithms (MD5, SHA1 for passwords)
- Missing HTTPS enforcement / HSTS headers

**A03: Injection**
- Raw SQL queries with string concatenation/interpolation
- Unparameterized queries in custom data access code
- LDAP injection in directory lookups
- OS command injection via `Process.Start` or `exec` with user input
- XSS via unencoded output (`@Html.Raw()` with user content, `dangerouslySetInnerHTML` with unsanitized data)
- Template injection in server-side rendering

**A04: Insecure Design**
- Missing rate limiting on login/API endpoints
- No account lockout after failed attempts
- Sensitive data in URL query parameters
- Missing input validation on file uploads (type, size)
- Unrestricted file upload paths

**A05: Security Misconfiguration**
- Debug mode enabled in production configs (`<compilation debug="true">`)
- Default credentials or admin paths unchanged
- Verbose error messages exposed to users (`<customErrors mode="Off">`)
- Unnecessary HTTP methods enabled
- Missing security headers (CSP, X-Frame-Options, X-Content-Type-Options)

**A06: Vulnerable and Outdated Components**
- See dependency scanning (Step 3)

**A07: Identification and Authentication Failures**
- Weak password policies (no complexity, short minimum)
- Session tokens in URLs
- Missing session timeout configuration
- No multi-factor support on admin interfaces

**A08: Software and Data Integrity Failures**
- Deserialization of untrusted data (`BinaryFormatter`, `JavaScriptSerializer` with type handling)
- Missing integrity checks on CDN resources (no SRI hash)
- Auto-update mechanisms without signature verification

**A09: Security Logging and Monitoring Failures**
- No audit logging on authentication events
- Sensitive data in log output
- Missing error logging configuration

**A10: Server-Side Request Forgery (SSRF)**
- User-supplied URLs passed to `HttpClient` without validation
- Webhook/callback URLs not validated against allowlist
- Image proxy or URL preview features without URL sanitization

### 3. Scan Dependencies

**NuGet (.NET):**
```bash
dotnet list package --vulnerable --include-transitive
```

If the CLI is not available, manually check `.csproj` files:
- Extract all `<PackageReference>` elements with versions
- Flag known-vulnerable packages and outdated major versions
- Check for deprecated packages (`EPiServer.*` → should be `Optimizely.*`)

**npm (Node.js):**
```bash
npm audit --json
```

If the CLI is not available, read `package-lock.json` and flag:
- Packages with known CVEs based on version ranges
- Outdated major versions of security-critical packages

### 4. Scan Configuration

Read CMS-specific security patterns from [cms-security.md](cms-security.md).

Check all configuration files for:
- Exposed secrets (connection strings, API keys, tokens)
- Overly permissive CORS policies
- Missing authentication on API endpoints
- Debug/development settings in production configs
- Insecure default values

### 5. Report

```
# Security Audit Results

**Platform:** [detected CMS]
**Scope:** code | deps | config | all
**Files scanned:** [count]

## Critical Vulnerabilities

### [A03] SQL Injection
**file:line** — Raw SQL with string interpolation
```csharp
// Vulnerable
var query = $"SELECT * FROM Content WHERE Id = {userInput}";

// Fix: Use parameterized queries
var query = "SELECT * FROM Content WHERE Id = @id";
cmd.Parameters.AddWithValue("@id", userInput);
```

## High Risk

### [A01] Missing Authorization
**file:line** — API endpoint has no `[Authorize]` attribute
Fix: Add `[Authorize(Roles = "Admin")]` or appropriate policy

## Medium Risk

...

## Dependency Vulnerabilities

| Package | Current | Severity | CVE | Fix Version |
|---------|---------|----------|-----|-------------|
| Example.Package | 3.1.0 | High | CVE-2024-XXXX | 3.1.5 |

## Configuration Issues

| File | Issue | Severity | Recommendation |
|------|-------|----------|----------------|
| appsettings.json | API key in plaintext | Critical | Move to environment variables or secret manager |

## Summary

| Category | Critical | High | Medium | Low |
|----------|----------|------|--------|-----|
| OWASP | X | X | X | X |
| CMS-specific | X | X | X | X |
| Dependencies | X | X | X | X |
| Configuration | X | X | X | X |

**Top 3 actions:**
1. ...
2. ...
3. ...
```

### Severity Levels

- **Critical** — Actively exploitable. Data breach, RCE, or auth bypass. Fix immediately.
- **High** — Exploitable with moderate effort. Privilege escalation, injection.
- **Medium** — Requires specific conditions to exploit. Information disclosure, misconfiguration.
- **Low** — Defense-in-depth issue. Missing headers, minor information leakage.
