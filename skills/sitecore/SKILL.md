---
name: sitecore
description: Sitecore XM Cloud and XP/XM expertise. Provides best practices for content modeling, component development, serialization, headless architecture with Content SDK, and SXA patterns. Auto-invoked when working in Sitecore projects.
user-invocable: false
allowed-tools: Read, Grep, Glob
---

# Sitecore XM Cloud & XP/XM Expert

You are a Sitecore specialist with deep knowledge of both XM Cloud (headless-first, Content SDK, Next.js) and traditional XP/XM (.NET MVC) development. Apply the guidance below when working in any Sitecore project.

## Architecture Overview

### XM Cloud (Headless-First)

XM Cloud is Sitecore's SaaS CMS. The architecture separates the content management backend from the presentation layer.

- **Content Management**: Hosted by Sitecore in the cloud. Authors use the Pages editor and Content Editor.
- **Content SDK / JSS SDK**: The bridge between the CMS and the front-end. Content SDK is the modern replacement for JSS, providing typed access to layout data, fields, and placeholders.
- **Next.js Rendering Host**: The head. A Next.js application fetches layout data from the Experience Edge or Layout Service and renders components.
- **Experience Edge**: CDN-delivered GraphQL endpoint serving published content.
- **SXA (Sitecore Experience Accelerator)**: Provides site scaffolding, component variants, grid system, and multi-site support out of the box.

### XP/XM Traditional (.NET MVC)

- **Sitecore Kernel**: The core platform running on .NET Framework or .NET Core (XM/XP 10.x).
- **Controller Renderings**: C# controllers return views for each component.
- **View Renderings**: `.cshtml` files bound directly to a datasource item.
- **Glass Mapper / ORM**: Optional mapping layer between Sitecore items and C# models.
- **Helix Architecture**: Recommended solution structure organized into Foundation, Feature, and Project layers.

## Content Modeling Rules

### Template Inheritance Hierarchy

Follow a strict hierarchy to maximize reuse and maintain consistency:

```
Base Templates (interfaces)
  e.g., _HasTitle, _HasImage, _HasMetadata
    └── Feature Templates (composed from bases)
          e.g., Article Feature, Hero Feature
            └── Page Templates (final, assigned to items)
                  e.g., Article Page, Landing Page
```

- **Base templates** define a single concern (one section with a small set of related fields). Prefix with underscore by convention: `_HasTitle`, `_HasSocialMeta`.
- **Feature templates** compose multiple base templates via inheritance. These represent a component's data shape.
- **Page templates** inherit from feature templates and any page-level bases. These are assigned to actual content items.

### Standard Values

Every template must have a Standard Values item (`__Standard Values`). Configure:

- **Insert Options**: Which child item types are allowed.
- **Default field values**: Sensible defaults for every field.
- **Presentation details**: Default layout and renderings so new items are immediately renderable.
- **Workflow**: Assign the initial workflow and workflow state.

Never skip Standard Values. They are not optional; they are the primary mechanism for consistent content governance.

### Field Naming Conventions

| Rule | Example |
|---|---|
| Use PascalCase for field names | `PageTitle`, `HeroImage`, `NavigationLink` |
| Use descriptive, unambiguous names | `ArticlePublishDate` not `Date` |
| Prefix boolean fields with `Is` or `Has` | `IsFeature`, `HasSidebar` |
| Group related fields into template sections | Section: `Content`, Section: `Navigation` |

### Field Types and When to Use Each

| Field Type | Use When |
|---|---|
| **Single-Line Text** | Short text values: titles, headings, labels |
| **Multi-Line Text** | Plain text without formatting: descriptions, alt text |
| **Rich Text** | Formatted content with HTML: body copy, paragraphs |
| **Image** | Image references from the Media Library |
| **General Link** | Internal links, external URLs, media links, anchors |
| **Date** / **Datetime** | Date or date-and-time values |
| **Checkbox** | Boolean true/false toggles |
| **Droptree** | Single item reference where the author picks from a tree |
| **Droplink** | Single item reference from a flat list |
| **Multilist** | Multiple item references from a flat list |
| **Treelist** | Multiple item references picked from a tree structure |
| **Multilist with Search** | Multiple references when the source set is very large |
| **Number** | Integer values |
| **Name Value List** | Key-value pairs (rendering parameters, structured data) |

**Selection guidance:**
- Prefer **Multilist** over **Treelist** unless the author needs to navigate a deep tree hierarchy.
- Prefer **Droptree** over **Droplink** when the source items are organized in folders.
- Use **General Link** rather than a plain text field for URLs so authors get link validation and internal link tracking.

### Template Sections

Organize fields into logical sections within each template:

- `Content` -- Primary content fields (title, body, image)
- `Navigation` -- Navigation-related fields (nav title, show in nav)
- `Metadata` -- SEO and social metadata fields
- `Settings` -- Configuration flags and options
- `Design` -- Layout or style override fields

Keep sections small and focused. If a section has more than 8 fields, consider splitting it or extracting a base template.

## Component Development

### Content SDK Components (Next.js -- Modern)

Content SDK is the current-generation approach for XM Cloud front-ends. Components are standard React functional components that receive layout data as props.

Key imports:
- `ComponentRendering` -- the rendering data for this component
- `useSitecoreContext` -- access page-level context (route data, site info)
- `Text`, `RichText`, `Image`, `Link` -- field helper components for inline editing
- `Placeholder` -- renders a named placeholder for nested components

See `patterns.md` for full code examples.

### JSS Components (Legacy)

Older JSS-based projects may use:
- `ComponentPropsContext` for prop injection
- `withDatasourceCheck` HOC to enforce datasource presence
- `useComponentProps` hook

These still work but new XM Cloud projects should use Content SDK patterns.

### Rendering Parameters

Rendering parameters let authors configure component behavior without a datasource item. Define them on the rendering item's `Parameters Template` field.

- Use rendering parameters for presentation concerns: column count, color theme, alignment.
- Use datasource items for content: text, images, links.
- Access in Content SDK via `props.rendering.params`.

### Component Variants with SXA

SXA component variants allow multiple visual representations of the same component without code changes:

- Variants are defined under `/sitecore/content/<site>/Presentation/Rendering Variants/<component>`.
- Each variant contains a set of variant fields that control output structure.
- Authors select variants via the rendering's `Variant` rendering parameter.
- In headless mode, variant selection is handled through rendering parameters and front-end component logic.

## Serialization (SCS)

Sitecore Content Serialization (SCS) is the standard for tracking Sitecore items in source control. It replaces TDS and Unicorn.

### Core Concepts

- **`sitecore.json`**: Root configuration at the solution root. References module files.
- **Module files (`*.module.json`)**: Define which items to serialize per logical module.
- **Scopes**: `SingleItem`, `ItemAndChildren`, `ItemAndDescendants`, `DescendantsOnly`.
- **Push operations**: `CreateAndUpdate`, `CreateUpdateAndDelete`.

### Workflow

```bash
dotnet sitecore cloud login          # Authenticate with XM Cloud
dotnet sitecore ser pull              # Pull items from Sitecore into files
dotnet sitecore ser push              # Push serialized files into Sitecore
dotnet sitecore ser validate          # Validate serialization config
dotnet sitecore ser diff              # Show differences between disk and Sitecore
```

See `serialization.md` for detailed configuration reference.

## Anti-Patterns to Avoid

### Do Not Put Content in Code

Never hardcode text, URLs, or image paths in component code. Always use datasource items so that content is editable by authors in the CMS.

```tsx
// WRONG
const Hero = () => <h1>Welcome to Our Site</h1>;

// RIGHT
const Hero = (props: HeroProps) => <Text field={props.fields.Heading} />;
```

### Do Not Hardcode GUIDs

Never reference Sitecore item IDs (GUIDs) directly in code. Use:
- Configuration items and site settings
- Named queries or relative paths
- Constants files that map logical names to IDs (as a last resort, with a comment explaining why)

### Do Not Skip Standard Values

Every template must have Standard Values configured. Skipping them leads to inconsistent item creation, missing presentation details, and broken workflows.

### Do Not Create Monolithic Templates

Break large templates into smaller base templates using inheritance. A template with 30+ fields is a sign that it should be decomposed:

```
BAD:  Article Page (30 fields in one template)
GOOD: Article Page inherits _HasTitle, _HasAuthor, _HasBody, _HasImage, _HasMetadata
```

### Do Not Use Treelist When Multilist Suffices

Treelist loads the entire content tree which is expensive. If the source items are in a flat list or a single folder, use Multilist or Multilist with Search.

### Do Not Nest Placeholders Too Deeply

Keep placeholder nesting to 3 levels or fewer. Deep nesting makes pages fragile and difficult for authors to manage. Flatten with sibling placeholders instead.

### Do Not Serialize Content Items in Development Modules

Serialization modules for templates and renderings should not include authored content. Keep content in a separate module with appropriate scope restrictions so that developer pulls do not overwrite author changes.

## Reference Files

- **`patterns.md`** -- Detailed code examples for Content SDK components, SXA patterns, MVC patterns, and GraphQL queries.
- **`serialization.md`** -- Full SCS configuration reference with module examples and CLI commands.
