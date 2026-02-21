---
name: cms-detect
description: Auto-detects the CMS platform (Sitecore, Umbraco, or Optimizely) in the current project by scanning project files, dependencies, and configuration. Invoked automatically when working in a CMS project.
user-invocable: false
allowed-tools: Read, Grep, Glob, Bash
---

# CMS Platform Detection

You are a CMS detection specialist. When working in a project, determine which CMS platform is in use and its configuration details.

## Detection Strategy

Scan the project in this order. Stop at the first definitive match.

### 1. Sitecore Detection

**Definitive signals:**
- `sitecore.json` at project root (SCS configuration)
- `.sitecore/` directory
- `*.module.json` serialization module files
- NuGet reference to `Sitecore.*` packages in `.csproj`

**Version detection:**
- `sitecore.json` with `"modules"` → XM Cloud / SCS
- Reference to `Sitecore.ContentSDK` → XM Cloud with Content SDK
- Reference to `Sitecore.LayoutService` → Headless (JSS/Content SDK)
- Reference to `Sitecore.Kernel` → XP/XM traditional
- `next.config.*` + Sitecore references → Next.js headless front-end

**Report:**
```
CMS: Sitecore
Edition: XM Cloud | XP | XM
Architecture: Headless (Content SDK / JSS) | Traditional (MVC)
Frontend: Next.js | React | None
Serialization: SCS (Sitecore Content Serialization)
```

### 2. Umbraco Detection

**Definitive signals:**
- NuGet reference to `Umbraco.Cms.*` in `.csproj`
- `appsettings.json` containing `"Umbraco"` configuration section
- `umbraco/` folder with `Data/` or `Models/` subfolders
- `Views/` folder with `.cshtml` files using `@inherits Umbraco.*`

**Version detection:**
- `Umbraco.Cms.Web.Common` version `14.*` → v14
- `Umbraco.Cms.Web.Common` version `15.*` → v15
- `Umbraco.Cms.Web.Common` version `16.*` → v16
- `Umbraco.Cms.Web.Common` version `17.*` → v17 LTS
- `App_Plugins/` with Lit/Web Components → Bellissima backoffice (v14+)

**Report:**
```
CMS: Umbraco
Version: 14 | 15 | 16 | 17
Backoffice: Bellissima (v14+) | AngularJS (legacy)
API: Content Delivery API | Management API
Content Sync: uSync | None detected
```

### 3. Optimizely Detection

**Definitive signals:**
- NuGet reference to `EPiServer.*` or `Optimizely.*` in `.csproj`
- `module.config` referencing EPiServer modules
- `appsettings.json` containing `"EPiServer"` or `"Optimizely"` section

**Version detection:**
- `Optimizely.CMS.Core` version `12.*` → CMS 12
- REST API usage with `https://*.cms.optimizely.com` → CMS SaaS
- Reference to `Optimizely.ContentGraph.Cms` → Using Graph API
- `next.config.*` or JS SDK references → Headless frontend

**Report:**
```
CMS: Optimizely
Edition: CMS 12 (.NET) | CMS SaaS
API: REST API | Graph API | Both
Frontend: .NET MVC | Next.js (JS SDK) | React
```

### 4. No CMS Detected

If no CMS is found, report:
```
CMS: None detected
Hint: This may be a frontend-only project or a new project. Use /scaffold to generate CMS-specific code.
```

## Usage

After detection, store the result mentally and use it to inform all subsequent CMS-related guidance. Other skills in this plugin (sitecore, umbraco, optimizely, scaffold, content-model) depend on this detection to provide platform-specific advice.
