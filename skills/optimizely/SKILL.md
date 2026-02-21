---
name: optimizely
description: Optimizely CMS 12 and CMS SaaS expertise. Provides best practices for content types, blocks, pages, Visual Builder, Graph API, REST API, experimentation integration, and .NET/headless development patterns. Auto-invoked when working in Optimizely projects.
user-invocable: false
allowed-tools: Read, Grep, Glob
---

# Optimizely CMS Expert

You are an Optimizely CMS expert covering both CMS 12 (.NET) and CMS SaaS. You provide guidance on content modeling, component development, API integration, and architectural patterns for traditional .NET MVC and headless implementations.

## Architecture Overview

### CMS 12 (.NET)

CMS 12 is built on .NET 6+ and runs as a self-hosted or DXP-hosted application.

- **Runtime:** ASP.NET Core (.NET 6, .NET 8)
- **Traditional MVC:** Razor views, PageControllers, ViewComponents, partial views
- **Headless option:** Content Delivery API or custom API controllers exposing content as JSON
- **Database:** SQL Server for content store, scheduled jobs, and configuration
- **Package namespace:** NuGet packages under `Optimizely.CMS.*` (legacy: `EPiServer.*`)
- **Content Delivery API:** Built-in REST endpoints for headless delivery when `Optimizely.ContentDeliveryApi.*` packages are added
- **Graph integration:** Install `Optimizely.ContentGraph.Cms` to sync content into Optimizely Graph for GraphQL queries

### CMS SaaS

CMS SaaS is the cloud-native, fully managed Optimizely CMS.

- **API-first:** All content operations happen through REST API and Graph API
- **Visual Builder:** Drag-and-drop page composition in the SaaS editorial UI
- **Graph API:** GraphQL-based content delivery (Optimizely Graph)
- **REST API:** Full CRUD for content and content types at `https://{instance}.cms.optimizely.com`
- **Authentication:** OAuth2 with client credentials or API keys
- **No .NET hosting required:** The CMS is fully managed; frontends are decoupled

### JS SDK (Headless Frontends)

The Optimizely JS SDK enables React/Next.js frontends to render CMS SaaS content.

- **Package:** `@optimizely/cms` and related packages
- **Visual Builder support:** Components register as Visual Builder elements
- **Content delivery:** Fetches content via Graph API or REST API
- **On-page editing:** Supports in-context editing within the SaaS editorial UI

## Content Modeling

### Content Types

Optimizely uses three primary content type categories:

| Category | CMS 12 Base Class | Purpose |
|----------|-------------------|---------|
| Page types | `PageData` | Routable pages with URLs |
| Block types | `BlockData` | Reusable content components placed in ContentAreas |
| Media types | `MediaData` (`ImageData`, `VideoData`) | Uploaded files and assets |

### Properties and Property Types

Common property types for CMS 12 content types:

| .NET Type | Editorial UI | Usage |
|-----------|-------------|-------|
| `string` | Text field | Short text, titles |
| `XhtmlString` | Rich text editor | HTML body content |
| `ContentArea` | Drag-and-drop area | Compose multiple blocks/pages |
| `ContentReference` | Content picker | Single reference to another content item |
| `Url` | URL picker | Internal or external links |
| `int`, `double`, `bool` | Numeric/toggle fields | Simple values |
| `IList<ContentReference>` | Multiple content picker | List of references |
| `DateTime` | Date picker | Date/time values |
| `PageReference` | Page picker | Reference specifically to a page |

### Content Type Attributes

```csharp
[ContentType(
    GUID = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    DisplayName = "Article Page",
    Description = "A standard article page with hero and body content",
    GroupName = "Content"
)]
[AvailableContentTypes(IncludeOn = new[] { typeof(StartPage) })]
public class ArticlePage : PageData
{
    // properties
}
```

Key attributes:
- `[ContentType]` — Registers the type with the CMS. `GUID` must be unique and stable.
- `[Display(Name, Description, GroupName, Order)]` — Controls editorial UI label, help text, tab, and sort order.
- `[Required]` — Makes the property mandatory in the editorial UI.
- `[CultureSpecific]` — Makes the property localizable per language branch.
- `[Searchable]` — Includes the property in full-text search indexing.
- `[Ignore]` — Excludes the property from the editorial UI.
- `[ScaffoldColumn(false)]` — Hides the property from UI but keeps it in the model.
- `[AvailableContentTypes]` — Restricts which parent types this content can be created under.
- `[AllowedTypes]` — Restricts allowed types on a ContentArea property.

### Property Groups (Tabs)

Properties are organized into tabs using `GroupName` in the `[Display]` attribute.

Built-in tabs via `SystemTabNames`:
- `SystemTabNames.Content` — Main content tab
- `SystemTabNames.Settings` — Settings tab
- `SystemTabNames.PageHeader` — Page header tab

Custom tab example:

```csharp
[GroupDefinitions]
public static class CustomTabNames
{
    [Display(Name = "SEO", Order = 300)]
    public const string SEO = "SEO";

    [Display(Name = "Hero", Order = 100)]
    public const string Hero = "Hero";
}
```

Usage on a property:

```csharp
[Display(Name = "Meta Description", GroupName = CustomTabNames.SEO, Order = 10)]
public virtual string MetaDescription { get; set; }
```

### ContentArea with Restricted Types
```csharp
[Display(Name = "Main Content Area", GroupName = SystemTabNames.Content, Order = 200)]
[AllowedTypes(typeof(TextBlock), typeof(ImageBlock), typeof(VideoBlock))]
public virtual ContentArea MainContentArea { get; set; }
```

### Validation Attributes
```csharp
[Required]
[StringLength(150, ErrorMessage = "Title cannot exceed 150 characters")]
[RegularExpression(@"^[a-zA-Z0-9\s]+$", ErrorMessage = "Only alphanumeric characters")]
public virtual string Title { get; set; }
```

### SaaS Content Type Definitions

In CMS SaaS, content types are defined via JSON through the REST API:

```json
{
  "key": "ArticlePage",
  "baseType": "Page",
  "displayName": "Article Page",
  "description": "A standard article page",
  "properties": {
    "title": {
      "type": "String",
      "displayName": "Title",
      "required": true,
      "localized": true
    },
    "body": {
      "type": "RichText",
      "displayName": "Body Content",
      "localized": true
    },
    "heroImage": {
      "type": "ContentReference",
      "displayName": "Hero Image",
      "allowedTypes": ["Image"]
    },
    "contentArea": {
      "type": "ContentArea",
      "displayName": "Content Area",
      "allowedTypes": ["TextBlock", "ImageBlock"]
    }
  }
}
```

## Component and Block Development

### CMS 12: Partial Views for Blocks

The simplest rendering approach. Create a partial view at `Views/Shared/Blocks/{BlockTypeName}.cshtml`:

```cshtml
@model MyProject.Models.Blocks.HeroBlock

<section class="hero">
    <h1>@Model.Heading</h1>
    @Html.PropertyFor(m => m.BackgroundImage)
    <div class="hero__body">@Html.PropertyFor(m => m.Body)</div>
</section>
```

### CMS 12: ViewComponent for Blocks

For blocks needing logic or data fetching:

```csharp
[TemplateDescriptor(
    Inherited = false,
    TemplateTypeCategory = TemplateTypeCategories.MvcPartialComponent)]
public class HeroBlockComponent : BlockComponent<HeroBlock>
{
    protected override IViewComponentResult InvokeComponent(HeroBlock currentContent)
    {
        var viewModel = new HeroBlockViewModel(currentContent)
        {
            HasVideo = !string.IsNullOrEmpty(currentContent.VideoUrl)
        };
        return View("~/Views/Shared/Blocks/HeroBlock.cshtml", viewModel);
    }
}
```

### CMS 12: ContentArea Rendering and Display Options

Render a ContentArea in a Razor view:

```cshtml
@Html.PropertyFor(m => m.MainContentArea)
```

With custom display options:

```csharp
[ServiceConfiguration(typeof(DisplayModeFallbackProvider))]
public class DisplayModeProvider : DisplayModeFallbackProvider
{
    public override List<DisplayModeFallback> GetAll()
    {
        return new List<DisplayModeFallback>
        {
            new() { Name = "Full", Tag = "full", ExtraCssClass = "col-12" },
            new() { Name = "Half", Tag = "half", ExtraCssClass = "col-6" },
            new() { Name = "Third", Tag = "third", ExtraCssClass = "col-4" },
        };
    }
}
```

### CMS SaaS: React Components via JS SDK

```tsx
import { CmsComponent, CmsEditable } from "@optimizely/cms/components";

interface HeroBlockProps {
  heading: string;
  body: string;
  backgroundImage?: {
    url: string;
    altText: string;
  };
}

export const HeroBlock: CmsComponent<HeroBlockProps> = ({ data }) => {
  return (
    <CmsEditable as="section" className="hero">
      <CmsEditable as="h1" propertyName="heading">
        {data.heading}
      </CmsEditable>
      {data.backgroundImage && (
        <img src={data.backgroundImage.url} alt={data.backgroundImage.altText} />
      )}
      <CmsEditable as="div" propertyName="body" className="hero__body">
        <div dangerouslySetInnerHTML={{ __html: data.body }} />
      </CmsEditable>
    </CmsEditable>
  );
};

HeroBlock.displayName = "HeroBlock";
```

### CMS SaaS: Visual Builder Configuration

Register components so the Visual Builder knows how to render them:

```tsx
import { registerComponent } from "@optimizely/cms/registration";

registerComponent(HeroBlock, {
  key: "HeroBlock",
  displayName: "Hero Block",
  category: "Content",
});
```

## Page Types and Routing

### CMS 12: PageController Pattern

Each page type can have a dedicated controller:

```csharp
public class ArticlePageController : PageController<ArticlePage>
{
    public ActionResult Index(ArticlePage currentPage)
    {
        var viewModel = new ArticlePageViewModel(currentPage)
        {
            PublishedDate = currentPage.StartPublish?.ToString("MMMM dd, yyyy")
        };
        return View(viewModel);
    }
}
```

The view resides at `Views/ArticlePage/Index.cshtml`.

Routing is convention-based: the CMS resolves the URL to a page in the content tree, identifies its page type, and dispatches to the matching `PageController<T>`.

### CMS SaaS: Pages via REST API

Pages are created and managed through the REST API. See [api-reference.md](api-reference.md) for endpoint details.

## Key Patterns

### Initialization Modules

Used to run code at application startup:

```csharp
[InitializableModule]
[ModuleDependency(typeof(EPiServer.Web.InitializationModule))]
public class CustomInitializationModule : IInitializableModule
{
    public void Initialize(InitializationEngine context)
    {
        // Subscribe to events, register services, etc.
        var contentEvents = context.Locate.Advanced.GetInstance<IContentEvents>();
        contentEvents.PublishedContent += OnPublishedContent;
    }

    public void Uninitialize(InitializationEngine context)
    {
        var contentEvents = context.Locate.Advanced.GetInstance<IContentEvents>();
        contentEvents.PublishedContent -= OnPublishedContent;
    }

    private void OnPublishedContent(object sender, ContentEventArgs e)
    {
        // Handle publish event
    }
}
```

For `IConfigurableModule` (to configure services in the DI container):

```csharp
[InitializableModule]
public class DependencyConfig : IConfigurableModule
{
    public void ConfigureContainer(ServiceConfigurationContext context)
    {
        context.Services.AddSingleton<IMyService, MyService>();
    }

    public void Initialize(InitializationEngine context) { }
    public void Uninitialize(InitializationEngine context) { }
}
```

### Scheduled Jobs

```csharp
[ScheduledPlugIn(
    DisplayName = "Content Cleanup Job",
    Description = "Removes expired content items",
    GUID = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    DefaultEnabled = true)]
public class ContentCleanupJob : ScheduledJobBase
{
    private readonly IContentRepository _contentRepository;

    public ContentCleanupJob(IContentRepository contentRepository)
    {
        _contentRepository = contentRepository;
        IsStoppable = true;
    }

    public override string Execute()
    {
        int count = 0;
        // Perform work, check _stopSignaled periodically
        return $"Cleaned up {count} expired items.";
    }
}
```

### Content Events

Subscribe to content lifecycle events via `IContentEvents`:

- `CreatingContent` / `CreatedContent`
- `SavingContent` / `SavedContent`
- `PublishingContent` / `PublishedContent`
- `DeletingContent` / `DeletedContent`
- `MovingContent` / `MovedContent`
- `CheckingInContent` / `CheckedInContent`

### Selection Factories

Provide dropdown options in the editorial UI:

```csharp
public class ColorSelectionFactory : ISelectionFactory
{
    public IEnumerable<ISelectItem> GetSelections(ExtendedMetadata metadata)
    {
        return new List<SelectItem>
        {
            new() { Text = "Red", Value = "red" },
            new() { Text = "Blue", Value = "blue" },
            new() { Text = "Green", Value = "green" },
        };
    }
}
```

Usage on a property:

```csharp
[Display(Name = "Theme Color", GroupName = SystemTabNames.Settings, Order = 10)]
[SelectOne(SelectionFactoryType = typeof(ColorSelectionFactory))]
public virtual string ThemeColor { get; set; }
```

## Anti-Patterns

Avoid these common mistakes:

| Anti-Pattern | Why It Is Harmful | Correct Approach |
|-------------|-------------------|------------------|
| Using `ContentReference` when `ContentArea` is appropriate | Editors lose drag-and-drop composition and display option support | Use `ContentArea` when editors should compose from multiple blocks |
| Skipping `[CultureSpecific]` on localizable properties | Property values cannot be translated per language branch | Always add `[CultureSpecific]` to any property that varies by language |
| Hardcoding content references (`new ContentReference(42)`) | Breaks across environments, fragile to content tree changes | Use settings pages, configuration, or content resolution by path |
| Deeply nested content type hierarchies | Difficult to maintain, confusing inheritance of properties | Prefer flat hierarchies; use interfaces or composition for shared behavior |
| Putting business logic in content types | Content types should be data models only; logic in types violates SRP | Place logic in services, controllers, or ViewComponents |
| Using `ServiceLocator` instead of dependency injection | Creates hidden dependencies, hard to test, discouraged in .NET Core | Use constructor injection via ASP.NET Core DI |
| Not setting a stable GUID on content types | Content type identity can be lost during refactoring or migration | Always set an explicit, unchanging `GUID` in `[ContentType]` |
| Ignoring content versioning in API calls | Can overwrite draft content or publish unintended versions | Use appropriate version-aware API methods and check publish status |

## Related Reference Files

- **[patterns.md](patterns.md)** — Detailed code patterns for CMS 12 and CMS SaaS development
- **[api-reference.md](api-reference.md)** — REST API, Graph API, and CMS 12 core service API reference
