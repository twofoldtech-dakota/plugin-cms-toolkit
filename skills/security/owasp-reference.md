# OWASP Top 10 — CMS Code Patterns

Quick reference mapping OWASP Top 10 categories to concrete code patterns found in .NET CMS projects and React/Next.js frontends.

---

## A01: Broken Access Control

### Vulnerable patterns to grep for

```
# Missing [Authorize] on controllers
class.*Controller.*:.*\{         # Controllers without [Authorize] above
[AllowAnonymous]                  # Verify these are intentional

# CORS
.AllowAnyOrigin()                 # Overly permissive
.WithOrigins("*")                 # Same issue

# Missing anti-forgery
[HttpPost]                        # Without [ValidateAntiForgeryToken] nearby
SurfaceController                 # Umbraco: all POST methods need anti-forgery

# Direct object reference
Request.Query["id"]               # User input used to fetch records without ownership check
RouteData.Values["id"]            # Same risk
```

### Sitecore-specific
- Publishing endpoints without auth
- Preview API accessible externally
- Sitecore Identity Server misconfiguration

### Umbraco-specific
- Content Delivery API without API key when `PublicAccess: false` content exists
- Management API endpoints exposed without Backoffice auth
- Member-protected content accessible via direct URL

### Optimizely-specific
- Content Delivery API without proper CORS
- Admin API endpoints without `[Authorize]`
- Scheduled jobs executable via URL without auth

---

## A02: Cryptographic Failures

### Vulnerable patterns

```
# Hardcoded secrets
connectionString.*=.*"Server=       # Connection string in code
"ApiKey".*:.*"[a-zA-Z0-9]           # API keys in appsettings
password.*=.*"                       # Hardcoded passwords

# Weak crypto
MD5.Create()                         # Weak hash
SHA1.Create()                        # Weak for passwords
new DESCryptoServiceProvider()       # Weak encryption

# Missing HTTPS
http://                              # In production config URLs
RequireHttpsMetadata.*false          # JWT config bypassing HTTPS
```

---

## A03: Injection

### SQL Injection patterns

```
# String concatenation in queries
$"SELECT.*{                          # Interpolated SQL
"SELECT.*" + variable                # Concatenated SQL
string.Format("SELECT              # Formatted SQL
.FromSqlRaw($"                       # EF Core raw SQL with interpolation
```

### XSS patterns

```
# .NET Razor
@Html.Raw(Model.                     # Unencoded HTML output from CMS field
@Html.Raw(ViewBag.                   # Unencoded dynamic content

# React/Next.js
dangerouslySetInnerHTML.*{           # Check if input is sanitized
innerHTML.*=                         # Direct DOM manipulation

# Umbraco
@Model.Value("richText")            # Should use @Model.Value<IHtmlEncodedString>
```

### Command Injection patterns

```
Process.Start(.*variable             # OS command with user input
Runtime.exec(                        # Java-style (in custom tools)
child_process.exec(                  # Node.js
```

---

## A04: Insecure Design

### Patterns indicating missing security design

```
# No rate limiting
[HttpPost].*Login                    # Login without rate limit middleware
/api/                                # API endpoints without throttling

# Unrestricted upload
IFormFile                            # Check for type/size validation
.SaveAs(                             # Check path validation
Request.Files                        # Legacy file upload
```

---

## A05: Security Misconfiguration

### Configuration patterns

```
# Debug mode
<compilation debug="true"            # web.config
"DetailedErrors".*true               # appsettings
ASPNETCORE_ENVIRONMENT.*Development  # Env in production

# Error exposure
<customErrors mode="Off"             # Exposes stack traces
"DeveloperExceptionPage"             # In production middleware

# Missing headers (check Startup.cs / Program.cs)
# Absence of: UseHsts, UseHttpsRedirection, Content-Security-Policy
```

---

## A07: Authentication Failures

### Patterns

```
# Session config
.SessionTimeout.*=.*0               # No timeout
"CookieHttpOnly".*false             # Cookies accessible via JS
SameSite.*None                       # Check if necessary
Secure.*false                        # Cookies sent over HTTP

# Password policy
MinimumLength.*[1-5]"                # Weak minimum
RequireDigit.*false                  # Weak policy
RequireUppercase.*false              # Weak policy
```

---

## A08: Integrity Failures

### Deserialization patterns

```
# Dangerous deserializers
BinaryFormatter                      # Never use with untrusted data
JavaScriptSerializer.*TypeNameHandling  # Type confusion
JsonConvert.*TypeNameHandling.All    # Newtonsoft type handling
XmlSerializer.*user                  # XML with untrusted input

# Missing SRI
<script src="https://cdn             # External scripts without integrity=
<link.*href="https://cdn             # External CSS without integrity=
```

---

## A10: SSRF

### Patterns

```
# URL from user input
new HttpClient().GetAsync(userUrl    # Direct SSRF
WebRequest.Create(.*Request          # User-supplied URL
fetch(.*props.                       # React: URL from props without validation
Image.*src.*=.*user                  # Image proxy
```
