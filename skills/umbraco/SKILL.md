---
name: umbraco
description: Umbraco CMS v14-17 expertise. Provides best practices for document types, compositions, property editors, Content Delivery API, Management API, Bellissima backoffice extensions, and .NET development patterns. Auto-invoked when working in Umbraco projects.
user-invocable: false
allowed-tools: Read, Grep, Glob
---

# Umbraco CMS Expert Skill

You are an Umbraco CMS expert specializing in versions 14 through 17, with a primary focus on v17 LTS. You guide developers through both traditional server-rendered Razor approaches and modern headless architectures using the Content Delivery API. You understand the Bellissima backoffice, .NET integration patterns, and Umbraco-specific conventions that produce maintainable, scalable content platforms.

## Architecture Overview

### Runtime and Framework

Umbraco v14+ runs on modern .NET. The version mapping is:

| Umbraco Version | .NET Version | Support Level |
|-----------------|-------------|---------------|
| v14             | .NET 8      | STS           |
| v15             | .NET 9      | STS           |
| v16             | .NET 9      | STS           |
| v17             | .NET 10     | LTS           |

Umbraco v17 is the current Long Term Support release. Target it for new projects unless there is a specific reason to use an earlier version.

### Bellissima Backoffice (v14+)

Starting with v14, Umbraco replaced the AngularJS backoffice with Bellissima, a new backoffice built on:

- **Lit** web components and TypeScript
- A reactive state management layer
- The Management API as its backend (no more `UmbracoApiController` for backoffice)
- Extension points via manifests and web components

The backoffice is now a single-page application that communicates exclusively through the Management API. Custom backoffice extensions are authored as Lit-based web components registered through a manifest system (`umbraco-package.json`).

### Content Delivery API (Headless)

The Content Delivery API is a built-in, read-only REST API for delivering published content and media to front-end applications. It supports:

- Fetching content by route, ID, or content type
- Expansion of related content and media
- Filtering, sorting, and pagination
- Protected content with member-based authentication
- Output caching for performance

This API makes Umbraco viable as a headless CMS without any custom controller code.

### Traditional Razor Views

Umbraco continues to support server-rendered output through:

- Razor templates mapped to document types
- Strongly-typed models via ModelsBuilder
- Partial views and view components
- RenderController for custom routing logic

Both approaches (headless and Razor) can coexist in the same installation.

## Document Type Design

Document types are the schema definitions for content in Umbraco. Well-designed document types are the foundation of a maintainable Umbraco site.

### Compositions vs Inheritance

Umbraco supports both single-parent inheritance and compositions (mixins) for document types.

**Prefer compositions** for reusable field groups. Compositions allow a document type to include property groups from multiple sources without a rigid hierarchy.

Use compositions for:

- SEO fields (meta title, meta description, open graph image)
- Navigation properties (hide from navigation, navigation title)
- Header/footer configuration
- Social media links
- Common content fields shared across unrelated types

Use inheritance only when there is a true parent-child relationship where the child genuinely "is a" specialization of the parent, such as `ArticlePage` inheriting from a base `ContentPage`.

### Element Types

Element types are document types with the "Is an Element Type" flag set. They cannot be created as standalone content nodes. Use them for:

- Block List items
- Block Grid items
- Nested Content items (legacy, prefer Block List)
- Any structured content that lives within another document, not as its own node

Element types keep the content tree clean and provide structured, reusable content blocks.

### Property Editors

Choose the right property editor for each field:

| Editor | Use When |
|--------|----------|
| **TextBox** | Short single-line text (titles, labels, slugs) |
| **TextArea** | Multi-line plain text (descriptions, excerpts) |
| **Rich Text Editor** | Formatted content with HTML output |
| **Numeric** | Integer or decimal values |
| **True/False** | Boolean toggles |
| **Dropdown** | Selection from a fixed list of options |
| **Tags** | Free-form tagging |
| **MediaPicker** | Selecting one or more media items |
| **ContentPicker** | Selecting one or more content nodes |
| **MultiUrlPicker** | Links (internal, external, or media) |
| **Block List** | Ordered list of structured content blocks |
| **Block Grid** | Grid-based layout of structured content blocks |
| **Date Picker** | Date or date/time values |
| **Eye Dropper Color Picker** | Color selection |
| **Slider** | Numeric range selection |
| **Member Picker** | Selecting members |
| **Member Group Picker** | Selecting member groups |

### Naming Conventions

Follow these conventions for consistency:

- **Document Type alias**: camelCase (e.g., `articlePage`, `homePage`)
- **Document Type name**: Readable with spaces (e.g., `Article Page`, `Home Page`)
- **Property alias**: camelCase (e.g., `pageTitle`, `heroImage`, `bodyContent`)
- **Property label**: Readable with spaces (e.g., `Page Title`, `Hero Image`)
- **Composition alias**: camelCase with `Composition` suffix (e.g., `seoComposition`)
- **Composition name**: Readable with `Composition` suffix (e.g., `SEO Composition`)
- **Element Type alias**: camelCase (e.g., `heroBlock`, `testimonialCard`)
- **Tab names**: Short, descriptive (e.g., `Content`, `Settings`, `SEO`, `Navigation`)

### Tab Organization

Organize properties into logical tabs:

- **Content** tab: Primary editorial content (title, body, images)
- **Settings** tab: Behavioral settings (template override, hide from nav)
- **SEO** tab: Search engine optimization fields (from composition)
- **Navigation** tab: Navigation-related fields (from composition)
- **Media** tab: Media selections when there are many

Put the most frequently edited fields in the first tab. Editors should not need to hunt for common fields.

### Property Validation

Always apply validation where appropriate:

- Mark required fields as mandatory
- Set character limits on text fields that feed into constrained spaces (meta titles, social cards)
- Use regex validation for structured input (email, phone, postal code)
- Provide helpful validation messages that tell editors what is expected

## Content Delivery API

### Enabling and Configuring

Enable the Content Delivery API in `appsettings.json`:

```json
{
  "Umbraco": {
    "CMS": {
      "DeliveryApi": {
        "Enabled": true,
        "PublicAccess": true,
        "ApiKey": "your-optional-api-key"
      }
    }
  }
}
```

The API is available at `/umbraco/delivery/api/v2/content` once enabled.

### Expansion and Filtering

Use the `expand` parameter to include related content inline rather than as references. Use `filter` to narrow results by content type, name, or custom criteria. Use `sort` to order results.

Example: Fetch all `articlePage` items sorted by creation date:

```
/umbraco/delivery/api/v2/content?filter=contentType:articlePage&sort=createDate:desc&take=10
```

### Custom Output Models

Implement `IContentResponseModel` or use output expansion configuration to control the shape of API responses. This keeps the API output clean for front-end consumers.

### Protected Content Delivery

For member-protected content, configure the Delivery API to respect member authentication:

```json
{
  "Umbraco": {
    "CMS": {
      "DeliveryApi": {
        "MemberAuthorization": {
          "AuthorizationCodeFlow": {
            "Enabled": true,
            "LoginRedirectUrls": ["https://your-frontend.com/callback"],
            "LogoutRedirectUrls": ["https://your-frontend.com"]
          }
        }
      }
    }
  }
}
```

## Block List and Block Grid Patterns

### Block List

Use Block List for ordered sequences of content blocks where layout control is one-dimensional (vertical stacking). Common uses:

- Page body content with mixed block types
- FAQ sections (question/answer blocks)
- Feature lists
- Testimonial carousels

Each block in a Block List has a content model (an element type) and an optional settings model (another element type for presentation options like background color or spacing).

### Block Grid

Use Block Grid when editors need two-dimensional layout control (rows and columns). Common uses:

- Landing page builders
- Multi-column layouts
- Dashboard-style pages

Block Grid adds layout configuration (column spans, row placement) on top of content and settings models.

### When to Choose

- **Block List**: Content order matters, layout is linear, simpler editing experience needed
- **Block Grid**: Editors need column/grid control, layout is two-dimensional, landing-page-style flexibility required

## Controller Patterns

### RenderController

Extend `RenderController` to customize how a content node is rendered. Use it to:

- Add view model data beyond what the content node provides
- Implement custom routing logic per document type
- Redirect based on content state

### SurfaceController

Extend `SurfaceController` for form handling and postback scenarios. It provides:

- Built-in anti-forgery token support
- Access to `UmbracoHelper` and `IPublishedContent`
- Standard MVC form posting patterns

### ApiController

Use standard ASP.NET `ApiController` (or `ControllerBase`) for custom API endpoints that do not need Umbraco context. For endpoints that need Umbraco services, inject them through constructor injection.

### UmbracoAuthorizedApiController (Legacy)

In v14+, backoffice APIs are handled through the Management API. Custom backoffice API endpoints should be implemented as Management API extensions rather than using the legacy `UmbracoAuthorizedApiController`.

## Key Anti-Patterns

Avoid these common mistakes:

### Do Not Use Inheritance When Compositions Work

Document type inheritance creates rigid hierarchies. If `ArticlePage` and `ProductPage` both need SEO fields, do not create a `BasePage` parent just for those fields. Use an `SeoComposition` instead. Reserve inheritance for true is-a relationships.

### Do Not Put Business Logic in Views

Razor views should render data, not compute it. Move logic into:

- Controllers (RenderController for per-request logic)
- Services (registered in DI)
- View components

### Do Not Hardcode Content IDs

Never reference content by integer ID or hardcoded GUID in code. Instead:

- Use content pickers on document types to let editors choose related content
- Use `IPublishedContentQuery` to find content by type or property
- Use configuration or constants for well-known content keys

### Do Not Skip Property Validation

Every field that feeds a constrained UI element (meta titles, alt text, slugs) must have appropriate validation. Editors will enter unexpected values if validation does not guide them.

### Do Not Create God Document Types

A document type with 40+ properties is a maintenance burden. Break it apart:

- Use compositions for cross-cutting field groups
- Use Block List or Block Grid for variable content sections
- Create focused document types that represent a single concept

## Related References

For detailed code patterns, see [patterns.md](./patterns.md).

For API reference documentation, see [api-reference.md](./api-reference.md).
