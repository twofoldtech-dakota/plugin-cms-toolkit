---
name: scaffold
description: Scaffolds CMS components, content types, controllers, and page types for Sitecore, Umbraco, or Optimizely. Detects the CMS platform automatically and generates platform-specific code. Usage - /plugin-cms-toolkit:scaffold component HeroBlock
user-invocable: true
argument-hint: "<type> <name> [options]"
allowed-tools: Read, Write, Grep, Glob, Bash, Edit
---

# CMS Scaffold Generator

You generate production-ready code for CMS projects. You detect the CMS platform first, then generate platform-specific files.

## Usage

```
/plugin-cms-toolkit:scaffold <type> <name>
```

**Types:** `component`, `content-type`, `controller`, `page`

**Examples:**
```
/plugin-cms-toolkit:scaffold component HeroBanner
/plugin-cms-toolkit:scaffold content-type ArticlePage
/plugin-cms-toolkit:scaffold controller Search
/plugin-cms-toolkit:scaffold page LandingPage
```

## Step 1: Detect CMS

Before generating, determine the CMS platform:

1. Look for `sitecore.json` or `Sitecore.*` NuGet refs → **Sitecore**
2. Look for `Umbraco.Cms.*` NuGet refs or Umbraco config → **Umbraco**
3. Look for `Optimizely.*` or `EPiServer.*` NuGet refs → **Optimizely**
4. If unclear, ask the user which CMS to target

Also determine the architecture:
- **Headless**: `next.config.*`, JS SDK refs, Content SDK → generate frontend component + CMS definition
- **Traditional .NET**: MVC controllers, Razor views → generate C# code + views

## Step 2: Scan Project Conventions

Before generating files, scan the existing codebase to match conventions:
- Find existing components/controllers to match file organization and coding style
- Check namespace patterns in existing `.cs` files
- Check import patterns in existing `.tsx`/`.ts` files
- Match existing indentation (tabs vs spaces, width)
- Follow existing folder structure

## Step 3: Generate

### `component` — UI Component with CMS Integration

**Sitecore (Headless / Content SDK):**
- `src/components/<Name>.tsx` — React component with Content SDK field helpers
- Props interface with typed Sitecore fields
- Uses `Text`, `RichText`, `Image`, `Link` from `@sitecore-jss/sitecore-jss-nextjs` or `@sitecore/content-sdk`
- Includes `ComponentRendering` prop typing

**Sitecore (Traditional MVC):**
- `Feature/<Feature>/code/Controllers/<Name>Controller.cs` — Controller rendering
- `Feature/<Feature>/code/Models/<Name>Model.cs` — Glass Mapper model
- `Feature/<Feature>/code/Views/<Name>/<Name>.cshtml` — Razor view

**Umbraco (Razor):**
- `Views/Partials/<Name>.cshtml` — Partial view using ModelsBuilder types
- For blocks: `Views/Partials/blocklist/Components/<Name>.cshtml`
- Strong-typed with `@inherits UmbracoViewPage<BlockListItem>`

**Umbraco (Headless):**
- Frontend component file (React/Next.js) consuming Content Delivery API
- Type definitions matching the Umbraco content structure

**Optimizely (CMS 12):**
- `Models/Blocks/<Name>Block.cs` — Block content type with `[ContentType]` attribute
- `Views/Shared/Blocks/<Name>Block.cshtml` — Razor partial view
- Or `Components/<Name>BlockViewComponent.cs` for ViewComponent pattern

**Optimizely (SaaS / Headless):**
- React component using JS SDK
- Content type JSON definition for REST API registration

### `content-type` — Content Model Definition

**Sitecore:**
- Serialization item (`.yml`) for the template under `/sitecore/templates/`
- Template sections with fields
- Standard Values item
- `__Standard values` serialization entry
- Adds include rule to the appropriate `*.module.json`

**Umbraco:**
- `Models/<Name>.cs` — ModelsBuilder-compatible class (if SourceCodeManual)
- Document type registration via composition pattern
- Suggests property editors for common fields
- Includes property validation attributes

**Optimizely (CMS 12):**
- `Models/Pages/<Name>Page.cs` or `Models/Blocks/<Name>Block.cs`
- `[ContentType]` attribute with GUID, DisplayName, Description, GroupName
- Properties with `[Display]` attributes organized by tabs
- Appropriate field types (`XhtmlString`, `ContentArea`, `ContentReference`, etc.)

**Optimizely (SaaS):**
- JSON content type definition for REST API
- Properties with types, display names, localization settings

### `controller` — Backend Controller

**Sitecore (Traditional):**
- `Controllers/<Name>Controller.cs` — inherits `SitecoreController`
- Index action with datasource item access
- ViewModel class

**Umbraco:**
- `Controllers/<Name>Controller.cs` — `RenderController` or `SurfaceController`
- For forms: `SurfaceController` with `[HttpPost]`, `[ValidateAntiForgeryToken]`
- For API: `UmbracoApiController` with route attribute
- ViewModel if needed

**Optimizely (CMS 12):**
- `Controllers/<Name>PageController.cs` — inherits `PageController<T>`
- ViewModel with `CurrentPage` property mapping
- For API: standard `ApiController` with content service injection

### `page` — Full Page Type with Template

**Sitecore:**
- Template serialization YAML with all sections (Content, Hero, SEO, Metadata)
- Standard Values with default layout and presentation details
- Content SDK page component (`src/components/<Name>Page.tsx`) or MVC view
- Route entry if needed

**Umbraco:**
- Document type class with common page compositions (SEO, OpenGraph, Navigation)
- Matching Razor template (`Views/<Name>.cshtml`) with layout
- Master template integration

**Optimizely (CMS 12):**
- Page type class (`Models/Pages/<Name>Page.cs`) with standard page properties
- Controller (`Controllers/<Name>PageController.cs`)
- View (`Views/<Name>Page/Index.cshtml`) with layout
- ViewModel class

**Optimizely (SaaS):**
- JSON content type definition (baseType: "page")
- React page component via JS SDK

## Step 4: Post-Generation

After generating files:
1. List all created files with brief descriptions
2. Note any manual steps needed (e.g., "Register this template in Sitecore", "Run `dotnet sitecore ser push`")
3. Suggest related scaffolds (e.g., after creating a content-type, suggest creating a matching component)

## Naming Rules

Always follow CMS-specific naming conventions:
- **Sitecore:** PascalCase for templates and fields, component file matches rendering name
- **Umbraco:** camelCase for aliases, PascalCase for class names, readable labels
- **Optimizely:** PascalCase for class names and properties, suffix types with `Page`/`Block`/`File`
