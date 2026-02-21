# CMS Toolkit

> Sitecore, Umbraco, and Optimizely expertise baked into Claude Code — scaffolding, content modeling, code review, and best practices enforcement for agency teams.

<!-- ![License: MIT](https://img.shields.io/badge/license-MIT-blue) -->

## 30-Second Setup

```bash
# Add the marketplace source
/plugin marketplace add twofoldtech-dakota/cms-toolkit

# Install the plugin
/plugin install cms-toolkit@twofoldtech-dakota-cms-toolkit
```

No configuration needed. The plugin auto-detects your CMS platform from project files.

Open a Sitecore, Umbraco, or Optimizely project and start working — the toolkit activates automatically.

---

## What's Included

| Component | Name | What It Does |
|-----------|------|-------------|
| Skill | `/cms-toolkit:scaffold` | Generates components, content types, controllers, and pages |
| Skill | `/cms-toolkit:content-model` | Designs and audits content architectures |
| Skill | `cms-detect` | Auto-detects your CMS platform (runs in background) |
| Skill | `sitecore` | Deep Sitecore XM Cloud / XP expertise (auto-invoked) |
| Skill | `umbraco` | Deep Umbraco v14–17 expertise (auto-invoked) |
| Skill | `optimizely` | Deep Optimizely CMS 12 / SaaS expertise (auto-invoked) |
| Agent | `cms-reviewer` | CMS-specific code review with anti-pattern detection |
| Hook | Pattern enforcement | Catches hardcoded GUIDs, secrets, N+1 queries on every edit |

---

## Usage

### Scaffold Code

Generate platform-specific, production-ready code that matches your project's existing conventions:

```
/cms-toolkit:scaffold component HeroBanner
/cms-toolkit:scaffold content-type ArticlePage
/cms-toolkit:scaffold controller Search
/cms-toolkit:scaffold page LandingPage
```

The scaffold skill:
1. Detects your CMS and architecture (headless vs traditional)
2. Scans your codebase for naming, namespace, and folder conventions
3. Generates files that fit right in
4. Lists created files and any manual follow-up steps

### Design Content Models

Plan content architectures interactively or from a description:

```
/cms-toolkit:content-model blog with categories and authors
```

Outputs a visual content type diagram with field mappings, then generates `/scaffold` commands to build it:

```
[Page Type] Article Page
├── Tab: Content
│   ├── Title (short text, required)
│   ├── Hero Image (image, required)
│   ├── Body (rich text)
│   └── Author (content reference → Author)
├── Tab: SEO (inherited from SEO Base)
│   ├── Meta Title (short text)
│   └── Meta Description (text area)
└── Tab: Settings
    └── Hide from Navigation (boolean)
```

Audit an existing content model for issues:

```
/cms-toolkit:content-model audit
```

### Background Expertise

Three platform skills activate automatically when you're working in a CMS project. They provide:

- **Content modeling rules** — template inheritance, compositions, field type selection
- **Component patterns** — Content SDK, Razor views, React/JS SDK, ViewComponents
- **API guidance** — Content Delivery API, Graph API, REST API, serialization
- **Anti-pattern prevention** — what not to do and why, with correct alternatives

### Code Review

The `cms-reviewer` agent checks for CMS-specific issues across all three platforms:

- Hardcoded content IDs and GUIDs
- Business logic in views
- N+1 query patterns
- Missing null checks on content references
- Platform-specific violations (ServiceLocator usage, wrong API for read vs write, missing localization attributes)

### Live Pattern Enforcement

A `PreToolUse` hook runs on every `Edit` and `Write` operation, catching common mistakes before they land:

- Hardcoded GUIDs or content IDs
- Secrets written directly in code
- Business logic in Razor/JSX views
- Content queries inside loops
- Missing null checks on content references

---

## Supported Platforms

| Platform | Versions | Architectures |
|----------|----------|---------------|
| **Sitecore** | XM Cloud, XP, XM | Headless (Content SDK / JSS + Next.js), Traditional (.NET MVC) |
| **Umbraco** | v14, v15, v16, v17 LTS | Razor views, Content Delivery API (headless) |
| **Optimizely** | CMS 12, CMS SaaS | .NET MVC, Headless (REST API, Graph API, JS SDK) |

---

## How It Works

```
cms-toolkit/
├── .claude-plugin/
│   └── plugin.json              # Plugin metadata
├── skills/
│   ├── cms-detect/              # Auto-detection (background)
│   ├── sitecore/                # Sitecore expertise + patterns + serialization ref
│   ├── umbraco/                 # Umbraco expertise + patterns + API ref
│   ├── optimizely/              # Optimizely expertise + patterns + API ref
│   ├── scaffold/                # /scaffold command
│   └── content-model/           # /content-model command
├── agents/
│   └── cms-reviewer.md          # CMS-aware code reviewer
└── hooks/
    └── hooks.json               # Pattern enforcement
```

The plugin is entirely declarative — markdown skills, a JSON hook, and an agent definition. No compiled code, no runtime dependencies.

---

## Troubleshooting

**Problem:** The plugin doesn't detect my CMS platform.
**Fix:** Ensure you're in the project root directory with `.csproj` files or `sitecore.json` visible. The detection scans for NuGet package references and platform-specific config files.

**Problem:** Scaffold generates code that doesn't match my project style.
**Fix:** The scaffold skill scans existing files for conventions. If your project is empty or has very few files, it falls back to platform defaults. Add a few reference files first, then scaffold.

**Problem:** The pattern enforcement hook flags code I think is fine.
**Fix:** The hook is intentionally strict. If a specific pattern is acceptable in your context, approve the edit when prompted. The hook checks for common anti-patterns, not absolute rules.

**Problem:** I'm using a CMS version that isn't listed.
**Fix:** The toolkit covers Sitecore XM Cloud/XP/XM, Umbraco v14–17, and Optimizely CMS 12/SaaS. Older versions (Umbraco v13 and below, Optimizely CMS 11) may partially work but are not explicitly supported.

---

## License

MIT
