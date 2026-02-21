# Sitecore Content Serialization (SCS) Reference

Complete reference for Sitecore Content Serialization configuration, module structure, and CLI commands.

## Root Configuration: sitecore.json

The `sitecore.json` file lives at the solution root and defines the serialization environment.

```json
{
  "$schema": ".sitecore/schemas/RootConfigurationFile.schema.json",
  "modules": [
    "src/Foundation/*/*.module.json",
    "src/Feature/*/*.module.json",
    "src/Project/*/*.module.json"
  ],
  "serialization": {
    "defaultMaxRelativeItemPathLength": 100,
    "defaultModuleRelativeSerializationPath": "items"
  },
  "plugins": [
    {
      "resolve": "Sitecore.DevEx.Extensibility.Publishing"
    },
    {
      "resolve": "Sitecore.DevEx.Extensibility.Serialization"
    },
    {
      "resolve": "Sitecore.DevEx.Extensibility.Indexing"
    }
  ]
}
```

### Key Properties

| Property | Description |
|---|---|
| `modules` | Glob patterns pointing to module configuration files. Order matters for dependency resolution. |
| `serialization.defaultMaxRelativeItemPathLength` | Maximum length of serialized file paths relative to the module. Increase if you have deeply nested items. |
| `serialization.defaultModuleRelativeSerializationPath` | Default folder name within each module for serialized `.yml` files. Typically `items`. |
| `plugins` | Sitecore CLI plugins to load. Serialization plugin is required. |

## Module Files: *.module.json

Each module file defines a logical grouping of items to serialize. Place module files alongside the project they belong to (following Helix conventions).

### Full Module Structure

```json
{
  "namespace": "Feature.Articles",
  "items": {
    "includes": [
      {
        "name": "templates",
        "path": "/sitecore/templates/Feature/Articles",
        "scope": "ItemAndDescendants",
        "allowedPushOperations": "CreateUpdateAndDelete"
      },
      {
        "name": "renderings",
        "path": "/sitecore/layout/Renderings/Feature/Articles",
        "scope": "ItemAndDescendants",
        "allowedPushOperations": "CreateUpdateAndDelete"
      },
      {
        "name": "placeholder-settings",
        "path": "/sitecore/layout/Placeholder Settings/Feature/Articles",
        "scope": "ItemAndDescendants",
        "allowedPushOperations": "CreateUpdateAndDelete"
      },
      {
        "name": "media",
        "path": "/sitecore/media library/Feature/Articles",
        "scope": "ItemAndDescendants",
        "allowedPushOperations": "CreateAndUpdate"
      }
    ]
  }
}
```

### Module Properties

| Property | Description |
|---|---|
| `namespace` | Unique identifier for the module. Should match the project name (e.g., `Feature.Articles`). |
| `items.includes` | Array of include rules defining which items to serialize. |

## Include Rules

Each entry in the `includes` array defines a set of items to track.

### Properties

| Property | Required | Description |
|---|---|---|
| `name` | Yes | Unique name within the module. Used as the folder name for serialized files. |
| `path` | Yes | The Sitecore item path to include. |
| `scope` | No | How deep to include. Default: `ItemAndDescendants`. |
| `allowedPushOperations` | No | What operations are allowed on push. Default: `CreateAndUpdate`. |
| `rules` | No | Array of child rules for fine-grained control. |

### Scope Values

| Scope | Behavior |
|---|---|
| `SingleItem` | Only the item at the specified path. No children. |
| `ItemAndChildren` | The item and its direct children only (one level deep). |
| `ItemAndDescendants` | The item and all descendants at any depth. Most common scope. |
| `DescendantsOnly` | All descendants but not the item itself. Use when the parent folder is managed by another module. |
| `Ignored` | Explicitly exclude this path. Used in child rules to skip subtrees. |

### Allowed Push Operations

| Value | Behavior |
|---|---|
| `CreateOnly` | Items can be created but not updated or deleted. Protects existing content. |
| `CreateAndUpdate` | Items can be created and updated but not deleted. Safe default for content. |
| `CreateUpdateAndDelete` | Full control. Items can be created, updated, and deleted. Use for developer-owned items like templates and renderings. |

### Child Rules for Fine-Grained Control

Use the `rules` array within an include to override scope or operations for specific subtrees.

```json
{
  "name": "content",
  "path": "/sitecore/content/Site/Home",
  "scope": "ItemAndDescendants",
  "allowedPushOperations": "CreateAndUpdate",
  "rules": [
    {
      "path": "/articles",
      "scope": "ItemAndChildren",
      "allowedPushOperations": "CreateOnly"
    },
    {
      "path": "/settings",
      "scope": "ItemAndDescendants",
      "allowedPushOperations": "CreateUpdateAndDelete"
    },
    {
      "path": "/temp",
      "scope": "Ignored"
    }
  ]
}
```

Rule paths are relative to the parent include's `path`. In the example above, `/articles` resolves to `/sitecore/content/Site/Home/articles`.

## Common Module Configurations

### Templates Module

Tracks all templates owned by a feature. Use `CreateUpdateAndDelete` since templates are developer-owned.

```json
{
  "namespace": "Feature.Navigation",
  "items": {
    "includes": [
      {
        "name": "templates",
        "path": "/sitecore/templates/Feature/Navigation",
        "scope": "ItemAndDescendants",
        "allowedPushOperations": "CreateUpdateAndDelete"
      },
      {
        "name": "branches",
        "path": "/sitecore/templates/Branches/Feature/Navigation",
        "scope": "ItemAndDescendants",
        "allowedPushOperations": "CreateUpdateAndDelete"
      }
    ]
  }
}
```

### Renderings Module

Tracks rendering definitions and placeholder settings.

```json
{
  "namespace": "Feature.Hero",
  "items": {
    "includes": [
      {
        "name": "renderings",
        "path": "/sitecore/layout/Renderings/Feature/Hero",
        "scope": "ItemAndDescendants",
        "allowedPushOperations": "CreateUpdateAndDelete"
      },
      {
        "name": "placeholder-settings",
        "path": "/sitecore/layout/Placeholder Settings/Feature/Hero",
        "scope": "ItemAndDescendants",
        "allowedPushOperations": "CreateUpdateAndDelete"
      },
      {
        "name": "rendering-parameters",
        "path": "/sitecore/templates/Feature/Hero/Rendering Parameters",
        "scope": "ItemAndDescendants",
        "allowedPushOperations": "CreateUpdateAndDelete"
      }
    ]
  }
}
```

### Content Module

Tracks authored content. Use `CreateAndUpdate` (never delete) to avoid wiping out author work.

```json
{
  "namespace": "Project.Website.Content",
  "items": {
    "includes": [
      {
        "name": "content-root",
        "path": "/sitecore/content/Site",
        "scope": "SingleItem",
        "allowedPushOperations": "CreateOnly"
      },
      {
        "name": "content-home",
        "path": "/sitecore/content/Site/Home",
        "scope": "ItemAndDescendants",
        "allowedPushOperations": "CreateAndUpdate",
        "rules": [
          {
            "path": "/Data",
            "scope": "ItemAndDescendants",
            "allowedPushOperations": "CreateAndUpdate"
          }
        ]
      },
      {
        "name": "content-settings",
        "path": "/sitecore/content/Site/Settings",
        "scope": "ItemAndDescendants",
        "allowedPushOperations": "CreateUpdateAndDelete"
      },
      {
        "name": "dictionary",
        "path": "/sitecore/content/Site/Dictionary",
        "scope": "ItemAndDescendants",
        "allowedPushOperations": "CreateAndUpdate"
      }
    ]
  }
}
```

### Media Module

Tracks media library items. Use `CreateAndUpdate` for authored media. Use `CreateUpdateAndDelete` only for developer-owned icons or design assets.

```json
{
  "namespace": "Project.Website.Media",
  "items": {
    "includes": [
      {
        "name": "media-project",
        "path": "/sitecore/media library/Project/Website",
        "scope": "ItemAndDescendants",
        "allowedPushOperations": "CreateAndUpdate"
      },
      {
        "name": "media-icons",
        "path": "/sitecore/media library/Project/Website/Icons",
        "scope": "ItemAndDescendants",
        "allowedPushOperations": "CreateUpdateAndDelete"
      }
    ]
  }
}
```

### Foundation Module (Shared Base Templates)

Tracks shared base templates and foundation-level items used across multiple features.

```json
{
  "namespace": "Foundation.SharedTemplates",
  "items": {
    "includes": [
      {
        "name": "base-templates",
        "path": "/sitecore/templates/Foundation/SharedTemplates",
        "scope": "ItemAndDescendants",
        "allowedPushOperations": "CreateUpdateAndDelete"
      }
    ]
  }
}
```

### Site Definition Module (XM Cloud)

For XM Cloud projects, the site definition and SXA configuration items must be serialized.

```json
{
  "namespace": "Project.Website.SiteDefinition",
  "items": {
    "includes": [
      {
        "name": "site-definition",
        "path": "/sitecore/content/Site",
        "scope": "SingleItem",
        "allowedPushOperations": "CreateAndUpdate"
      },
      {
        "name": "site-settings",
        "path": "/sitecore/content/Site/Settings",
        "scope": "ItemAndDescendants",
        "allowedPushOperations": "CreateUpdateAndDelete"
      },
      {
        "name": "site-presentation",
        "path": "/sitecore/content/Site/Presentation",
        "scope": "ItemAndDescendants",
        "allowedPushOperations": "CreateUpdateAndDelete"
      },
      {
        "name": "headless-variants",
        "path": "/sitecore/content/Site/Presentation/Headless Variants",
        "scope": "ItemAndDescendants",
        "allowedPushOperations": "CreateUpdateAndDelete"
      },
      {
        "name": "available-renderings",
        "path": "/sitecore/content/Site/Presentation/Available Renderings",
        "scope": "ItemAndDescendants",
        "allowedPushOperations": "CreateUpdateAndDelete"
      }
    ]
  }
}
```

## CLI Commands Reference

All commands use the Sitecore CLI (`dotnet sitecore`). Install via:

```bash
dotnet tool install Sitecore.CLI
dotnet sitecore plugin add -n Sitecore.DevEx.Extensibility.Serialization
```

### Authentication

```bash
# Login to XM Cloud
dotnet sitecore cloud login

# Login to a specific Sitecore instance (non-cloud)
dotnet sitecore login --authority https://id.example.com --cm https://cm.example.com --allow-write true

# Check login status
dotnet sitecore cloud environment list
```

### Serialization Pull

Pull items from a Sitecore instance to disk (YAML files).

```bash
# Pull all modules
dotnet sitecore ser pull

# Pull a specific module by namespace
dotnet sitecore ser pull --include "Feature.Articles"

# Pull with overwrite (replaces local files with server state)
dotnet sitecore ser pull --overwrite

# Pull in a specific environment (XM Cloud)
dotnet sitecore ser pull --environment-id <env-id>
```

When to pull:
- After making changes in the Sitecore Content Editor or Pages
- When onboarding to synchronize your local files with the shared environment
- Before committing to ensure your serialized items are current

### Serialization Push

Push serialized YAML files from disk into a Sitecore instance.

```bash
# Push all modules
dotnet sitecore ser push

# Push a specific module
dotnet sitecore ser push --include "Feature.Articles"

# Dry run (show what would change without applying)
dotnet sitecore ser push --dry-run

# Push in a specific environment (XM Cloud)
dotnet sitecore ser push --environment-id <env-id>
```

When to push:
- After pulling another developer's changes from source control
- When setting up a new Sitecore instance
- During CI/CD deployment

### Validation

```bash
# Validate serialization configuration files for syntax errors
dotnet sitecore ser validate

# Validate with verbose output
dotnet sitecore ser validate --verbose
```

Run validation:
- Before committing changes to catch configuration errors early
- In CI pipelines to prevent broken serialization from being merged

### Diff

```bash
# Show differences between local serialized files and the Sitecore instance
dotnet sitecore ser diff

# Diff a specific module
dotnet sitecore ser diff --include "Feature.Articles"
```

Use diff to understand what will change before running a push or pull.

### Package Generation (XM Cloud Deploy)

For XM Cloud, serialized items are automatically deployed via the XM Cloud build pipeline. The `sitecore.json` and module files are included in the deploy artifact.

```bash
# Create a serialization package for manual deployment
dotnet sitecore ser package create -o package.itempackage

# Install a package
dotnet sitecore ser package install -f package.itempackage
```

## Best Practices for Serialization Scope

### Separate Developer Items from Content

Developer-owned items (templates, renderings, placeholder settings) should use `CreateUpdateAndDelete` so that renames and removals propagate cleanly.

Content items (pages, data folders, dictionary entries) should use `CreateAndUpdate` or `CreateOnly` to prevent serialization from deleting author work.

```
Developer items → CreateUpdateAndDelete
Shared content  → CreateAndUpdate
Seed content    → CreateOnly
Author content  → Do not serialize (or CreateOnly for structure)
```

### One Module per Feature

Follow Helix and define one serialization module per Feature or Foundation project. This keeps modules focused and reduces merge conflicts.

```
src/
  Feature/
    Feature.Articles/
      Feature.Articles.module.json    ← templates + renderings for Articles
    Feature.Navigation/
      Feature.Navigation.module.json  ← templates + renderings for Navigation
  Foundation/
    Foundation.SharedTemplates/
      Foundation.SharedTemplates.module.json
  Project/
    Project.Website/
      Project.Website.module.json     ← site content, settings, media
```

### Avoid Overlapping Paths

Never have two modules include the same Sitecore path. Overlapping paths cause conflicts during push and unpredictable behavior. If two features share a parent folder, use `DescendantsOnly` on one or manage the parent in a Foundation module.

```json
// WRONG: Two modules both include /sitecore/templates/Feature
// Feature.Articles.module.json includes /sitecore/templates/Feature/Articles
// Feature.Hero.module.json includes /sitecore/templates/Feature/Hero
// Both are fine because they have distinct paths.

// WRONG: Overlapping
// Module A includes /sitecore/content/Site with ItemAndDescendants
// Module B includes /sitecore/content/Site/Home with ItemAndDescendants
// Module B's path is a subset of Module A. Remove it from one module.
```

### Keep Serialization Shallow for Content

For content items, serialize only what is needed for the site to function:

- **Structure items** (home, main sections): `SingleItem` or `ItemAndChildren`
- **Settings and configuration**: `ItemAndDescendants` with `CreateUpdateAndDelete`
- **Authored pages**: Either do not serialize, or use `CreateOnly` to seed structure without overwriting content

### Use Descriptive Include Names

Include names become folder names on disk. Use clear, descriptive names:

```json
// Good
{ "name": "templates", "path": "/sitecore/templates/Feature/Articles" }
{ "name": "renderings", "path": "/sitecore/layout/Renderings/Feature/Articles" }
{ "name": "placeholder-settings", "path": "/sitecore/layout/Placeholder Settings/Feature/Articles" }

// Bad
{ "name": "items1", "path": "/sitecore/templates/Feature/Articles" }
{ "name": "stuff", "path": "/sitecore/layout/Renderings/Feature/Articles" }
```

### Version Control Workflow

1. **Before starting work**: `dotnet sitecore ser pull` to get the latest state.
2. **Make changes** in Sitecore (Content Editor, Pages, or code).
3. **Pull changes**: `dotnet sitecore ser pull` to serialize your changes to disk.
4. **Review changes**: `git diff` to inspect serialized YAML files.
5. **Commit**: Add the serialized YAML files to your commit alongside code changes.
6. **After pulling from Git**: `dotnet sitecore ser push` to apply teammates' changes to your local Sitecore instance.
7. **Resolve conflicts**: If YAML files conflict in Git, resolve them in the YAML (or re-pull from Sitecore after merging in the CMS).
