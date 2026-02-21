# CMS-Specific Security Patterns

Security issues unique to each CMS platform. Check these in addition to OWASP Top 10.

---

## Sitecore

### Authentication and Authorization

| Issue | What to check | Severity |
|-------|---------------|----------|
| Sitecore admin exposed | `/sitecore/admin/` accessible externally | Critical |
| Identity Server misconfiguration | Token lifetimes, redirect URIs, client secrets in config | High |
| Publishing service unprotected | Publishing endpoints accessible without auth | High |
| Preview API open | Layout Service or Experience Edge preview endpoint public | Medium |
| Broken role inheritance | Security roles not propagating through template inheritance | Medium |

### Content and Data

| Issue | What to check | Severity |
|-------|---------------|----------|
| Hardcoded item GUIDs | GUIDs in code could be manipulated if exposed | Medium |
| Unsanitized Rich Text output | `@Html.Raw()` on rich text fields without sanitization | High |
| Media Library access control | Media items accessible by direct URL without auth check | Medium |
| GraphQL introspection enabled | Experience Edge schema exposed in production | Medium |
| Serialization overshare | SCS modules serializing sensitive items (security roles, users) | High |

### XM Cloud Specific

| Issue | What to check | Severity |
|-------|---------------|----------|
| Edge token exposure | Experience Edge API key in client-side code | Critical |
| Webhook secrets | Webhook endpoints without signature validation | High |
| Environment variables | Secrets in `next.config.js` without `NEXT_PUBLIC_` awareness | High |
| API route auth | Next.js API routes without proper authentication middleware | High |

---

## Umbraco

### Authentication and Authorization

| Issue | What to check | Severity |
|-------|---------------|----------|
| Backoffice path default | `/umbraco` accessible and not changed or IP-restricted | Medium |
| Management API exposed | Management API accessible outside backoffice context | Critical |
| Content Delivery API key missing | Public API without API key when serving protected content | High |
| Member auth bypass | Protected content accessible via Content Delivery API without member token | Critical |
| Default admin credentials | Unchanged default admin account after install | Critical |

### Configuration

| Issue | What to check | Severity |
|-------|---------------|----------|
| Unfiltered file uploads | Media upload without type/size restrictions | High |
| SMTP credentials in appsettings | Email server credentials in committed config | High |
| ModelsBuilder mode | `InMemoryAuto` in production (performance and potential info leak) | Low |
| Examine config exposed | Search index configuration revealing content structure | Low |

### Code Patterns

| Issue | What to check | Severity |
|-------|---------------|----------|
| `IContentService` in views | Write-capable service in presentation layer | Medium |
| Unvalidated redirect | `Redirect(Request.Query["returnUrl"])` without validation | High |
| SurfaceController without CSRF | POST actions missing `[ValidateAntiForgeryToken]` | High |
| UmbracoApiController auth | Custom API endpoints without `[Authorize]` | High |
| Raw content query injection | `IPublishedContentQuery` with unescaped user input | Medium |

### Content Delivery API

| Issue | What to check | Severity |
|-------|---------------|----------|
| Over-exposed properties | Sensitive properties visible in API output | High |
| Missing CORS config | API accessible from any origin | Medium |
| No rate limiting | API endpoints without throttling | Medium |
| Expansion depth unlimited | `expand=all` allowing deep content tree traversal | Medium |

---

## Optimizely

### Authentication and Authorization

| Issue | What to check | Severity |
|-------|---------------|----------|
| ServiceLocator usage | `ServiceLocator.Current` bypasses DI and auth context | Medium |
| Admin interface exposed | CMS admin panel accessible without VPN/IP restriction | High |
| API key in client code | REST API or Graph API key in frontend JavaScript | Critical |
| Missing `[Authorize]` on PageController | Custom controllers serving protected content without auth | High |

### CMS 12 Specific

| Issue | What to check | Severity |
|-------|---------------|----------|
| `IContentRepository` in views | Write-capable repository in Razor views | Medium |
| Find/Search injection | Unescaped user input in Optimizely Search queries | High |
| Visitor Group manipulation | Visitor group criteria evaluating client-supplied data without validation | Medium |
| Initialization module order | Security modules loading after content modules | Medium |
| Scheduled job secrets | Scheduled jobs with hardcoded credentials or connection strings | High |

### CMS SaaS Specific

| Issue | What to check | Severity |
|-------|---------------|----------|
| OAuth client secret exposure | Client credentials in frontend code or committed config | Critical |
| Webhook endpoint unprotected | Incoming webhooks without HMAC signature validation | High |
| Graph API overshare | GraphQL queries returning unpublished or internal content | High |
| REST API write access | Content management API keys with write scope in read-only contexts | Medium |

### Common Patterns

| Issue | What to check | Severity |
|-------|---------------|----------|
| `BinaryFormatter` deserialization | Custom serialization using unsafe deserializers | Critical |
| Content type GUID collision | Duplicate GUIDs across content types (integrity risk) | Medium |
| Event handler secrets | `IContentEvents` handlers with hardcoded connection info | Medium |
| Selection factory data leak | `ISelectionFactory` returning sensitive options to all users | Low |

---

## Cross-Platform Configuration Checks

These apply regardless of CMS:

| Check | File(s) | Severity |
|-------|---------|----------|
| Connection strings in committed files | `appsettings.json`, `web.config`, `.env` | Critical |
| API keys in source | `*.cs`, `*.ts`, `*.tsx`, `*.json` | Critical |
| HTTPS not enforced | `Startup.cs`, `Program.cs`, `next.config.*` | High |
| HSTS not configured | Middleware pipeline | Medium |
| CSP header missing | Middleware or `web.config` | Medium |
| X-Frame-Options missing | Clickjacking risk | Medium |
| X-Content-Type-Options missing | MIME sniffing risk | Low |
| Referrer-Policy missing | Information leakage | Low |
| `.env` not in `.gitignore` | `.gitignore` | Critical |
| Debug builds deployed | `.csproj` build config, `NODE_ENV` | High |
