# Optimizely CMS Code Patterns

Detailed code patterns for CMS 12 (.NET) and CMS SaaS development.

## CMS 12 Content Type Patterns

### Page Type with Properties, Attributes, and Tabs

```csharp
using System.ComponentModel.DataAnnotations;
using EPiServer.Core;
using EPiServer.DataAbstraction;
using EPiServer.DataAnnotations;
using EPiServer.Web;

[ContentType(
    GUID = "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    DisplayName = "Article Page",
    Description = "A page for articles with hero, body, and sidebar content",
    GroupName = "Content")]
[AvailableContentTypes(IncludeOn = new[] { typeof(StartPage), typeof(ContainerPage) })]
public class ArticlePage : PageData
{
    #region Hero Tab

    [Display(Name = "Hero Heading",
        Description = "Main heading displayed in the hero area",
        GroupName = CustomTabNames.Hero,
        Order = 10)]
    [CultureSpecific]
    [Required]
    [StringLength(120)]
    public virtual string HeroHeading { get; set; }

    [Display(Name = "Hero Image",
        Description = "Background image for the hero section",
        GroupName = CustomTabNames.Hero,
        Order = 20)]
    [UIHint(UIHint.Image)]
    public virtual ContentReference HeroImage { get; set; }

    [Display(Name = "Hero Subheading",
        GroupName = CustomTabNames.Hero,
        Order = 30)]
    [CultureSpecific]
    public virtual string HeroSubheading { get; set; }

    #endregion

    #region Content Tab

    [Display(Name = "Body Content",
        Description = "Main article body in rich text",
        GroupName = SystemTabNames.Content,
        Order = 100)]
    [CultureSpecific]
    [Required]
    public virtual XhtmlString Body { get; set; }

    [Display(Name = "Author",
        GroupName = SystemTabNames.Content,
        Order = 110)]
    [CultureSpecific]
    public virtual string Author { get; set; }

    [Display(Name = "Publish Date",
        GroupName = SystemTabNames.Content,
        Order = 120)]
    public virtual DateTime ArticleDate { get; set; }

    [Display(Name = "Tags",
        Description = "Comma-separated tags for categorization",
        GroupName = SystemTabNames.Content,
        Order = 130)]
    [CultureSpecific]
    public virtual IList<string> Tags { get; set; }

    [Display(Name = "Main Content Area",
        GroupName = SystemTabNames.Content,
        Order = 200)]
    [AllowedTypes(typeof(TextBlock), typeof(ImageBlock), typeof(VideoBlock), typeof(CallToActionBlock))]
    public virtual ContentArea MainContentArea { get; set; }

    #endregion

    #region Sidebar Tab

    [Display(Name = "Sidebar Content Area",
        GroupName = "Sidebar",
        Order = 300)]
    [AllowedTypes(typeof(TextBlock), typeof(CallToActionBlock))]
    public virtual ContentArea SidebarContentArea { get; set; }

    [Display(Name = "Related Articles",
        Description = "Links to related article pages",
        GroupName = "Sidebar",
        Order = 310)]
    [AllowedTypes(typeof(ArticlePage))]
    public virtual ContentArea RelatedArticles { get; set; }

    #endregion

    #region SEO Tab

    [Display(Name = "Meta Title",
        GroupName = CustomTabNames.SEO,
        Order = 400)]
    [CultureSpecific]
    [StringLength(70)]
    public virtual string MetaTitle { get; set; }

    [Display(Name = "Meta Description",
        GroupName = CustomTabNames.SEO,
        Order = 410)]
    [CultureSpecific]
    [StringLength(160)]
    public virtual string MetaDescription { get; set; }

    [Display(Name = "Open Graph Image",
        GroupName = CustomTabNames.SEO,
        Order = 420)]
    [UIHint(UIHint.Image)]
    public virtual ContentReference OpenGraphImage { get; set; }

    [Display(Name = "Hide from Search Engines",
        GroupName = CustomTabNames.SEO,
        Order = 430)]
    public virtual bool NoIndex { get; set; }

    #endregion

    #region Settings Tab

    [Display(Name = "Theme Color",
        GroupName = SystemTabNames.Settings,
        Order = 500)]
    [SelectOne(SelectionFactoryType = typeof(ColorSelectionFactory))]
    public virtual string ThemeColor { get; set; }

    [Display(Name = "Show Breadcrumbs",
        GroupName = SystemTabNames.Settings,
        Order = 510)]
    public virtual bool ShowBreadcrumbs { get; set; }

    #endregion
}
```

### Custom Tab Definitions

```csharp
using EPiServer.DataAnnotations;
using System.ComponentModel.DataAnnotations;

[GroupDefinitions]
public static class CustomTabNames
{
    [Display(Name = "Hero", Order = 50)]
    public const string Hero = "Hero";

    [Display(Name = "SEO", Order = 300)]
    public const string SEO = "SEO";

    [Display(Name = "Social Media", Order = 310)]
    public const string SocialMedia = "SocialMedia";

    [Display(Name = "Analytics", Order = 400)]
    public const string Analytics = "Analytics";
}
```

### Block Type with Rendering

Block model:

```csharp
[ContentType(
    GUID = "b2c3d4e5-f6a7-8901-bcde-f12345678901",
    DisplayName = "Hero Block",
    Description = "A hero banner with heading, body text, and call-to-action",
    GroupName = "Content Blocks")]
public class HeroBlock : BlockData
{
    [Display(Name = "Heading", GroupName = SystemTabNames.Content, Order = 10)]
    [CultureSpecific]
    [Required]
    public virtual string Heading { get; set; }

    [Display(Name = "Body Text", GroupName = SystemTabNames.Content, Order = 20)]
    [CultureSpecific]
    public virtual XhtmlString Body { get; set; }

    [Display(Name = "Background Image", GroupName = SystemTabNames.Content, Order = 30)]
    [UIHint(UIHint.Image)]
    public virtual ContentReference BackgroundImage { get; set; }

    [Display(Name = "CTA Text", GroupName = SystemTabNames.Content, Order = 40)]
    [CultureSpecific]
    public virtual string CallToActionText { get; set; }

    [Display(Name = "CTA Link", GroupName = SystemTabNames.Content, Order = 50)]
    public virtual Url CallToActionLink { get; set; }

    [Display(Name = "Layout Style", GroupName = SystemTabNames.Settings, Order = 100)]
    [SelectOne(SelectionFactoryType = typeof(HeroLayoutSelectionFactory))]
    public virtual string LayoutStyle { get; set; }
}
```

Razor partial view at `Views/Shared/Blocks/HeroBlock.cshtml`:

```cshtml
@model MyProject.Models.Blocks.HeroBlock

<section class="hero hero--@Model.LayoutStyle">
    @if (Model.BackgroundImage != null)
    {
        <img class="hero__bg" src="@Url.ContentUrl(Model.BackgroundImage)" alt="" />
    }
    <div class="hero__content">
        <h1 class="hero__heading">@Html.PropertyFor(m => m.Heading)</h1>
        <div class="hero__body">@Html.PropertyFor(m => m.Body)</div>
        @if (!string.IsNullOrEmpty(Model.CallToActionText))
        {
            <a class="hero__cta" href="@Url.ContentUrl(Model.CallToActionLink)">
                @Model.CallToActionText
            </a>
        }
    </div>
</section>
```

### Selection Factory Pattern

```csharp
using EPiServer.Shell.ObjectEditing;

public class HeroLayoutSelectionFactory : ISelectionFactory
{
    public IEnumerable<ISelectItem> GetSelections(ExtendedMetadata metadata)
    {
        return new List<SelectItem>
        {
            new() { Text = "Full Width", Value = "full-width" },
            new() { Text = "Centered", Value = "centered" },
            new() { Text = "Left Aligned", Value = "left-aligned" },
            new() { Text = "Split (Image Left)", Value = "split-left" },
            new() { Text = "Split (Image Right)", Value = "split-right" },
        };
    }
}
```

For multi-select (checkboxes):

```csharp
[Display(Name = "Visible On", GroupName = SystemTabNames.Settings, Order = 200)]
[SelectMany(SelectionFactoryType = typeof(DeviceSelectionFactory))]
public virtual string VisibleOnDevices { get; set; }

public class DeviceSelectionFactory : ISelectionFactory
{
    public IEnumerable<ISelectItem> GetSelections(ExtendedMetadata metadata)
    {
        return new List<SelectItem>
        {
            new() { Text = "Desktop", Value = "desktop" },
            new() { Text = "Tablet", Value = "tablet" },
            new() { Text = "Mobile", Value = "mobile" },
        };
    }
}
```

### ContentArea with Allowed Types

```csharp
[Display(Name = "Feature Cards", GroupName = SystemTabNames.Content, Order = 200)]
[AllowedTypes(typeof(FeatureCardBlock), typeof(TestimonialBlock))]
public virtual ContentArea FeatureCards { get; set; }
```

To restrict where a block type can be created:

```csharp
[ContentType(
    GUID = "...",
    DisplayName = "Feature Card Block",
    GroupName = "Content Blocks")]
[AvailableContentTypes(IncludeOn = new[] { typeof(LandingPage), typeof(HomePage) })]
public class FeatureCardBlock : BlockData
{
    // properties
}
```

### Media Type

```csharp
[ContentType(
    GUID = "c3d4e5f6-a7b8-9012-cdef-123456789012",
    DisplayName = "Image File",
    Description = "Image asset with alt text and copyright metadata")]
[MediaDescriptor(ExtensionString = "jpg,jpeg,png,gif,webp,svg")]
public class ImageFile : ImageData
{
    [Display(Name = "Alt Text", GroupName = SystemTabNames.Content, Order = 10)]
    [CultureSpecific]
    public virtual string AltText { get; set; }

    [Display(Name = "Copyright", GroupName = SystemTabNames.Content, Order = 20)]
    public virtual string Copyright { get; set; }

    [Display(Name = "Photographer", GroupName = SystemTabNames.Content, Order = 30)]
    public virtual string Photographer { get; set; }
}
```

Video media type:

```csharp
[ContentType(
    GUID = "d4e5f6a7-b8c9-0123-defa-234567890123",
    DisplayName = "Video File",
    Description = "Video asset with caption and transcript")]
[MediaDescriptor(ExtensionString = "mp4,webm,ogg")]
public class VideoFile : VideoData
{
    [Display(Name = "Caption", GroupName = SystemTabNames.Content, Order = 10)]
    [CultureSpecific]
    public virtual string Caption { get; set; }

    [Display(Name = "Transcript", GroupName = SystemTabNames.Content, Order = 20)]
    [CultureSpecific]
    public virtual XhtmlString Transcript { get; set; }

    [Display(Name = "Duration (seconds)", GroupName = SystemTabNames.Content, Order = 30)]
    public virtual int Duration { get; set; }
}
```

## CMS 12 Controller Patterns

### PageController with ViewModel

Controller:

```csharp
using EPiServer.Web.Mvc;
using Microsoft.AspNetCore.Mvc;

public class ArticlePageController : PageController<ArticlePage>
{
    private readonly IContentLoader _contentLoader;

    public ArticlePageController(IContentLoader contentLoader)
    {
        _contentLoader = contentLoader;
    }

    public ActionResult Index(ArticlePage currentPage)
    {
        var viewModel = new ArticlePageViewModel(currentPage)
        {
            FormattedDate = currentPage.ArticleDate.ToString("MMMM dd, yyyy"),
            HasSidebar = currentPage.SidebarContentArea != null
                         && currentPage.SidebarContentArea.FilteredItems.Any(),
        };

        if (currentPage.RelatedArticles != null)
        {
            viewModel.RelatedArticleItems = currentPage.RelatedArticles
                .FilteredItems
                .Select(item => _contentLoader.Get<ArticlePage>(item.ContentLink))
                .ToList();
        }

        return View(viewModel);
    }
}
```

ViewModel:

```csharp
public class ArticlePageViewModel
{
    public ArticlePage CurrentPage { get; }
    public string FormattedDate { get; set; }
    public bool HasSidebar { get; set; }
    public List<ArticlePage> RelatedArticleItems { get; set; } = new();

    public ArticlePageViewModel(ArticlePage currentPage)
    {
        CurrentPage = currentPage;
    }
}
```

View at `Views/ArticlePage/Index.cshtml`:

```cshtml
@model MyProject.ViewModels.ArticlePageViewModel

<article class="article">
    <header>
        <h1>@Html.PropertyFor(m => m.CurrentPage.HeroHeading)</h1>
        <p class="article__meta">
            By @Model.CurrentPage.Author | @Model.FormattedDate
        </p>
    </header>

    <div class="article__body">
        @Html.PropertyFor(m => m.CurrentPage.Body)
    </div>

    @Html.PropertyFor(m => m.CurrentPage.MainContentArea)

    @if (Model.HasSidebar)
    {
        <aside class="article__sidebar">
            @Html.PropertyFor(m => m.CurrentPage.SidebarContentArea)
        </aside>
    }
</article>
```

### BlockComponent / ViewComponent

For blocks that need business logic or additional data:

```csharp
using EPiServer.Web.Mvc;
using Microsoft.AspNetCore.Mvc;

[TemplateDescriptor(
    Inherited = false,
    TemplateTypeCategory = TemplateTypeCategories.MvcPartialComponent)]
public class CallToActionBlockComponent : BlockComponent<CallToActionBlock>
{
    private readonly IContentLoader _contentLoader;

    public CallToActionBlockComponent(IContentLoader contentLoader)
    {
        _contentLoader = contentLoader;
    }

    protected override IViewComponentResult InvokeComponent(CallToActionBlock currentContent)
    {
        var viewModel = new CallToActionBlockViewModel
        {
            Heading = currentContent.Heading,
            BodyHtml = currentContent.Body?.ToHtmlString(),
            ButtonText = currentContent.ButtonText,
            ButtonUrl = currentContent.ButtonLink?.ToString(),
            IsExternal = currentContent.ButtonLink?.IsAbsoluteUri == true,
        };

        return View("~/Views/Shared/Blocks/CallToActionBlock.cshtml", viewModel);
    }
}
```

Alternatively, use a standard ASP.NET Core ViewComponent:

```csharp
public class NavigationViewComponent : ViewComponent
{
    private readonly IContentLoader _contentLoader;

    public NavigationViewComponent(IContentLoader contentLoader)
    {
        _contentLoader = contentLoader;
    }

    public IViewComponentResult Invoke(ContentReference rootLink, int depth = 2)
    {
        var startPage = _contentLoader.Get<StartPage>(rootLink);
        var children = _contentLoader.GetChildren<PageData>(rootLink)
            .Where(p => p.VisibleInMenu)
            .ToList();

        var model = new NavigationViewModel
        {
            RootPage = startPage,
            TopLevelPages = children,
        };

        return View(model);
    }
}
```

### PartialContentController

Renders a specific content item within another page:

```csharp
public class PreviewArticleController : PartialContentController<ArticlePage>
{
    public override ActionResult Index(ArticlePage currentContent)
    {
        var model = new ArticlePreviewViewModel
        {
            Title = currentContent.HeroHeading,
            Summary = currentContent.MetaDescription,
            Url = currentContent.ContentLink,
        };
        return PartialView("~/Views/Shared/Partials/ArticlePreview.cshtml", model);
    }
}
```

### API Controller for Headless

Expose content through a custom API endpoint:

```csharp
using EPiServer;
using EPiServer.Core;
using Microsoft.AspNetCore.Mvc;

[ApiController]
[Route("api/[controller]")]
public class ArticlesApiController : ControllerBase
{
    private readonly IContentLoader _contentLoader;
    private readonly IContentRepository _contentRepository;

    public ArticlesApiController(
        IContentLoader contentLoader,
        IContentRepository contentRepository)
    {
        _contentLoader = contentLoader;
        _contentRepository = contentRepository;
    }

    [HttpGet]
    public ActionResult<IEnumerable<ArticleDto>> GetArticles(
        [FromQuery] int parentId,
        [FromQuery] int skip = 0,
        [FromQuery] int take = 10)
    {
        var parentRef = new ContentReference(parentId);
        var articles = _contentLoader.GetChildren<ArticlePage>(parentRef)
            .OrderByDescending(a => a.ArticleDate)
            .Skip(skip)
            .Take(take)
            .Select(a => new ArticleDto
            {
                Id = a.ContentLink.ID,
                Title = a.HeroHeading,
                Author = a.Author,
                Date = a.ArticleDate,
                MetaDescription = a.MetaDescription,
            });

        return Ok(articles);
    }

    [HttpGet("{id}")]
    public ActionResult<ArticleDto> GetArticle(int id)
    {
        var contentRef = new ContentReference(id);
        if (!_contentLoader.TryGet<ArticlePage>(contentRef, out var article))
        {
            return NotFound();
        }

        return Ok(new ArticleDto
        {
            Id = article.ContentLink.ID,
            Title = article.HeroHeading,
            Body = article.Body?.ToHtmlString(),
            Author = article.Author,
            Date = article.ArticleDate,
        });
    }
}
```

## CMS 12 Service Patterns

### IContentRepository Usage

`IContentRepository` provides full CRUD operations on content:

```csharp
public class ContentService
{
    private readonly IContentRepository _contentRepository;

    public ContentService(IContentRepository contentRepository)
    {
        _contentRepository = contentRepository;
    }

    // Get a content item by reference
    public T Get<T>(ContentReference reference) where T : IContentData
    {
        return _contentRepository.Get<T>(reference);
    }

    // Get children of a content item
    public IEnumerable<T> GetChildren<T>(ContentReference parentLink) where T : IContentData
    {
        return _contentRepository.GetChildren<T>(parentLink);
    }

    // Get ancestors (parent chain up to root)
    public IEnumerable<IContent> GetAncestors(ContentReference contentLink)
    {
        return _contentRepository.GetAncestors(contentLink);
    }

    // Create and save new content
    public ContentReference CreateArticle(ContentReference parentLink, string name, string heading)
    {
        var article = _contentRepository.GetDefault<ArticlePage>(parentLink);
        article.Name = name;
        article.HeroHeading = heading;
        article.ArticleDate = DateTime.Now;

        // SaveAction.Publish publishes immediately
        // SaveAction.ForLater saves as draft
        // SaveAction.CheckIn checks in for approval
        return _contentRepository.Save(article, SaveAction.Publish, AccessLevel.NoAccess);
    }

    // Update existing content
    public void UpdateArticle(ContentReference reference, string newHeading)
    {
        var article = _contentRepository.Get<ArticlePage>(reference);
        var writableClone = article.CreateWritableClone() as ArticlePage;
        writableClone.HeroHeading = newHeading;
        _contentRepository.Save(writableClone, SaveAction.Publish, AccessLevel.NoAccess);
    }

    // Move content to a new parent
    public void MoveContent(ContentReference contentLink, ContentReference newParent)
    {
        _contentRepository.Move(contentLink, newParent, AccessLevel.NoAccess, AccessLevel.NoAccess);
    }

    // Delete content (moves to trash)
    public void DeleteContent(ContentReference contentLink)
    {
        _contentRepository.Delete(contentLink, false); // false = move to trash
    }

    // Permanently delete (skip trash)
    public void PermanentlyDelete(ContentReference contentLink)
    {
        _contentRepository.Delete(contentLink, true); // true = permanent delete
    }
}
```

### IContentLoader for Read-Only Operations

`IContentLoader` is the read-only counterpart to `IContentRepository`. Prefer this when you only need to read content:

```csharp
public class ReadOnlyContentService
{
    private readonly IContentLoader _contentLoader;

    public ReadOnlyContentService(IContentLoader contentLoader)
    {
        _contentLoader = contentLoader;
    }

    public T Get<T>(ContentReference reference) where T : IContentData
    {
        return _contentLoader.Get<T>(reference);
    }

    public bool TryGet<T>(ContentReference reference, out T content) where T : IContentData
    {
        return _contentLoader.TryGet(reference, out content);
    }

    public IEnumerable<T> GetChildren<T>(ContentReference parentLink) where T : IContentData
    {
        return _contentLoader.GetChildren<T>(parentLink);
    }

    // Get content in a specific language
    public T GetInLanguage<T>(ContentReference reference, string languageCode) where T : IContentData
    {
        var languageSelector = new LanguageSelector(languageCode);
        return _contentLoader.Get<T>(reference, languageSelector);
    }

    // Get all language versions
    public IEnumerable<T> GetAllLanguageVersions<T>(ContentReference reference) where T : IContentData
    {
        return _contentLoader.GetItems(
            new[] { reference },
            new LoaderOptions { LanguageLoaderOption.FallbackWithMaster() })
            .OfType<T>();
    }
}
```

### IContentEvents for Event Handling

```csharp
[InitializableModule]
[ModuleDependency(typeof(EPiServer.Web.InitializationModule))]
public class ContentEventModule : IInitializableModule
{
    public void Initialize(InitializationEngine context)
    {
        var events = context.Locate.Advanced.GetInstance<IContentEvents>();

        events.PublishedContent += OnPublished;
        events.SavingContent += OnSaving;
        events.DeletingContent += OnDeleting;
        events.MovingContent += OnMoving;
    }

    public void Uninitialize(InitializationEngine context)
    {
        var events = context.Locate.Advanced.GetInstance<IContentEvents>();

        events.PublishedContent -= OnPublished;
        events.SavingContent -= OnSaving;
        events.DeletingContent -= OnDeleting;
        events.MovingContent -= OnMoving;
    }

    private void OnPublished(object sender, ContentEventArgs e)
    {
        if (e.Content is ArticlePage article)
        {
            // Trigger cache invalidation, send notifications, etc.
        }
    }

    private void OnSaving(object sender, ContentEventArgs e)
    {
        if (e.Content is ArticlePage article)
        {
            // Validate or transform content before save
            // Set e.CancelAction = true and e.CancelReason to block the save
        }
    }

    private void OnDeleting(object sender, DeleteContentEventArgs e)
    {
        // Prevent deletion of critical pages
        if (e.ContentLink.ID == 5) // example: start page
        {
            e.CancelAction = true;
            e.CancelReason = "Cannot delete the start page.";
        }
    }

    private void OnMoving(object sender, ContentEventArgs e)
    {
        // Log content moves for audit trail
    }
}
```

### Scheduled Job Pattern

```csharp
using EPiServer.PlugIn;
using EPiServer.Scheduler;
using EPiServer;
using EPiServer.Core;

[ScheduledPlugIn(
    DisplayName = "Expire Old Articles",
    Description = "Unpublishes articles older than the configured retention period",
    GUID = "e5f6a7b8-c9d0-1234-efab-345678901234",
    DefaultEnabled = false,
    SortIndex = 100)]
public class ExpireOldArticlesJob : ScheduledJobBase
{
    private readonly IContentRepository _contentRepository;
    private readonly IContentLoader _contentLoader;
    private bool _stopSignaled;

    public ExpireOldArticlesJob(
        IContentRepository contentRepository,
        IContentLoader contentLoader)
    {
        _contentRepository = contentRepository;
        _contentLoader = contentLoader;
        IsStoppable = true;
    }

    public override void Stop()
    {
        _stopSignaled = true;
    }

    public override string Execute()
    {
        int processed = 0;
        int expired = 0;
        var cutoffDate = DateTime.Now.AddDays(-365);

        var articlesRoot = new ContentReference(42); // configure via settings page
        var articles = _contentLoader.GetChildren<ArticlePage>(articlesRoot);

        foreach (var article in articles)
        {
            if (_stopSignaled)
            {
                return $"Job stopped. Processed {processed} articles, expired {expired}.";
            }

            processed++;

            if (article.ArticleDate < cutoffDate && article.Status == VersionStatus.Published)
            {
                var writable = article.CreateWritableClone() as ArticlePage;
                _contentRepository.Save(writable, SaveAction.RequestApproval, AccessLevel.NoAccess);
                expired++;
            }

            OnStatusChanged($"Processing article {processed}...");
        }

        return $"Completed. Processed {processed} articles, expired {expired}.";
    }
}
```

### Initialization Module Pattern

Basic initialization module:

```csharp
using EPiServer.Framework;
using EPiServer.Framework.Initialization;
using EPiServer.ServiceLocation;
using Microsoft.Extensions.DependencyInjection;

[InitializableModule]
[ModuleDependency(typeof(EPiServer.Web.InitializationModule))]
public class SiteInitializationModule : IInitializableModule
{
    public void Initialize(InitializationEngine context)
    {
        // Run at application startup
        // Subscribe to events, register custom logic
    }

    public void Uninitialize(InitializationEngine context)
    {
        // Run at application shutdown
        // Unsubscribe from events, clean up resources
    }
}
```

Configurable module (for DI registration):

```csharp
[InitializableModule]
public class DependencyRegistrationModule : IConfigurableModule
{
    public void ConfigureContainer(ServiceConfigurationContext context)
    {
        context.Services.AddScoped<IArticleService, ArticleService>();
        context.Services.AddSingleton<ICacheService, CacheService>();
        context.Services.AddTransient<IEmailService, EmailService>();
    }

    public void Initialize(InitializationEngine context) { }
    public void Uninitialize(InitializationEngine context) { }
}
```

## CMS SaaS Patterns

### Content Type Definition (JSON)

Create a page type via the REST API:

```json
{
  "key": "BlogPostPage",
  "baseType": "Page",
  "displayName": "Blog Post",
  "description": "A blog post with author, date, and rich content",
  "sortOrder": 100,
  "properties": {
    "title": {
      "type": "String",
      "displayName": "Title",
      "description": "The blog post title",
      "required": true,
      "localized": true,
      "sortOrder": 10
    },
    "subtitle": {
      "type": "String",
      "displayName": "Subtitle",
      "localized": true,
      "sortOrder": 20
    },
    "authorName": {
      "type": "String",
      "displayName": "Author Name",
      "sortOrder": 30
    },
    "publishDate": {
      "type": "DateTime",
      "displayName": "Publish Date",
      "sortOrder": 40
    },
    "heroImage": {
      "type": "ContentReference",
      "displayName": "Hero Image",
      "allowedTypes": ["Image"],
      "sortOrder": 50
    },
    "body": {
      "type": "RichText",
      "displayName": "Body Content",
      "localized": true,
      "sortOrder": 60
    },
    "mainContentArea": {
      "type": "ContentArea",
      "displayName": "Main Content Area",
      "allowedTypes": ["TextBlock", "ImageBlock", "VideoBlock"],
      "sortOrder": 70
    },
    "category": {
      "type": "String",
      "displayName": "Category",
      "sortOrder": 80
    },
    "featured": {
      "type": "Boolean",
      "displayName": "Featured Post",
      "sortOrder": 90
    }
  }
}
```

Block type definition:

```json
{
  "key": "TextBlock",
  "baseType": "Block",
  "displayName": "Text Block",
  "description": "A simple rich text block",
  "properties": {
    "heading": {
      "type": "String",
      "displayName": "Heading",
      "localized": true,
      "sortOrder": 10
    },
    "body": {
      "type": "RichText",
      "displayName": "Body",
      "localized": true,
      "sortOrder": 20
    }
  }
}
```

### Creating Content via REST API

Create a new content item:

```bash
POST https://{instance}.cms.optimizely.com/api/episerver/v3.0/content
Authorization: Bearer {access_token}
Content-Type: application/json

{
  "name": "My First Blog Post",
  "contentType": ["BlogPostPage"],
  "parentLink": {
    "id": 5
  },
  "language": {
    "name": "en"
  },
  "status": "Published",
  "title": {
    "value": "My First Blog Post"
  },
  "subtitle": {
    "value": "An introduction to our blog"
  },
  "authorName": {
    "value": "Jane Smith"
  },
  "publishDate": {
    "value": "2025-06-15T10:00:00Z"
  },
  "body": {
    "value": "<p>Welcome to our blog...</p>"
  },
  "featured": {
    "value": true
  }
}
```

Update existing content:

```bash
PATCH https://{instance}.cms.optimizely.com/api/episerver/v3.0/content/{contentId}
Authorization: Bearer {access_token}
Content-Type: application/json

{
  "title": {
    "value": "Updated Blog Post Title"
  },
  "status": "Published"
}
```

### Graph API Queries

Query blog posts from Optimizely Graph:

```graphql
query GetBlogPosts($locale: [Locales], $limit: Int) {
  BlogPostPage(
    locale: $locale
    limit: $limit
    orderBy: { publishDate: DESC }
    where: { featured: { eq: true } }
  ) {
    items {
      title
      subtitle
      authorName
      publishDate
      heroImage {
        url
      }
      body {
        html
      }
      category
      _link {
        default
      }
    }
    total
    cursor
  }
}
```

Query with filtering and facets:

```graphql
query SearchArticles($searchTerm: String, $category: String) {
  BlogPostPage(
    where: {
      _and: [
        { _fulltext: { match: $searchTerm } }
        { category: { eq: $category } }
      ]
    }
    orderBy: { _ranking: SEMANTIC }
  ) {
    items {
      title
      subtitle
      authorName
      publishDate
      _score
    }
    facets {
      category(limit: 10) {
        name
        count
      }
    }
    total
  }
}
```

### JS SDK Component Patterns (React)

Full page component with data fetching:

```tsx
import { CmsComponent, CmsEditable } from "@optimizely/cms/components";
import { useContent, useContentArea } from "@optimizely/cms/hooks";
import { ContentArea } from "@optimizely/cms/components";

interface BlogPostPageProps {
  title: string;
  subtitle?: string;
  authorName?: string;
  publishDate?: string;
  heroImage?: {
    url: string;
  };
  body?: string;
  mainContentArea?: any[];
}

export const BlogPostPage: CmsComponent<BlogPostPageProps> = ({ data }) => {
  const formattedDate = data.publishDate
    ? new Date(data.publishDate).toLocaleDateString("en-US", {
        year: "numeric",
        month: "long",
        day: "numeric",
      })
    : null;

  return (
    <article className="blog-post">
      <header className="blog-post__header">
        <CmsEditable as="h1" propertyName="title">
          {data.title}
        </CmsEditable>
        {data.subtitle && (
          <CmsEditable as="p" propertyName="subtitle" className="blog-post__subtitle">
            {data.subtitle}
          </CmsEditable>
        )}
        <div className="blog-post__meta">
          {data.authorName && <span>By {data.authorName}</span>}
          {formattedDate && <time>{formattedDate}</time>}
        </div>
      </header>

      {data.heroImage && (
        <CmsEditable propertyName="heroImage" className="blog-post__hero">
          <img src={data.heroImage.url} alt={data.title} />
        </CmsEditable>
      )}

      <CmsEditable propertyName="body" className="blog-post__body">
        <div dangerouslySetInnerHTML={{ __html: data.body || "" }} />
      </CmsEditable>

      <ContentArea
        propertyName="mainContentArea"
        items={data.mainContentArea}
        className="blog-post__content-area"
      />
    </article>
  );
};

BlogPostPage.displayName = "BlogPostPage";
```

Block component:

```tsx
import { CmsComponent, CmsEditable } from "@optimizely/cms/components";

interface TextBlockProps {
  heading?: string;
  body?: string;
}

export const TextBlock: CmsComponent<TextBlockProps> = ({ data }) => {
  return (
    <div className="text-block">
      {data.heading && (
        <CmsEditable as="h2" propertyName="heading">
          {data.heading}
        </CmsEditable>
      )}
      <CmsEditable propertyName="body" className="text-block__body">
        <div dangerouslySetInnerHTML={{ __html: data.body || "" }} />
      </CmsEditable>
    </div>
  );
};

TextBlock.displayName = "TextBlock";
```

### Visual Builder Configuration

Register all components with the Visual Builder:

```tsx
import { registerComponent } from "@optimizely/cms/registration";
import { BlogPostPage } from "./components/BlogPostPage";
import { TextBlock } from "./components/TextBlock";
import { ImageBlock } from "./components/ImageBlock";
import { VideoBlock } from "./components/VideoBlock";

// Register page components
registerComponent(BlogPostPage, {
  key: "BlogPostPage",
  displayName: "Blog Post Page",
  category: "Pages",
});

// Register block components
registerComponent(TextBlock, {
  key: "TextBlock",
  displayName: "Text Block",
  category: "Content Blocks",
});

registerComponent(ImageBlock, {
  key: "ImageBlock",
  displayName: "Image Block",
  category: "Media Blocks",
});

registerComponent(VideoBlock, {
  key: "VideoBlock",
  displayName: "Video Block",
  category: "Media Blocks",
});
```

## Naming Conventions

| Item | Convention | Example |
|------|-----------|---------|
| Page types | PascalCase, suffixed with `Page` | `ArticlePage`, `LandingPage`, `BlogPostPage` |
| Block types | PascalCase, suffixed with `Block` | `HeroBlock`, `TextBlock`, `CallToActionBlock` |
| Media types | PascalCase, suffixed with `File` or specific type | `ImageFile`, `VideoFile`, `DocumentFile` |
| Properties | PascalCase | `PageTitle`, `HeroHeading`, `MainContentArea` |
| Controllers | PascalCase, suffixed with `Controller` | `ArticlePageController`, `StartPageController` |
| ViewModels | PascalCase, suffixed with `ViewModel` | `ArticlePageViewModel`, `HeroBlockViewModel` |
| Selection factories | PascalCase, suffixed with `SelectionFactory` | `ColorSelectionFactory`, `LayoutSelectionFactory` |
| Scheduled jobs | PascalCase, suffixed with `Job` | `ContentCleanupJob`, `ExpireOldArticlesJob` |
| Initialization modules | PascalCase, suffixed with `Module` | `SiteInitializationModule`, `DependencyRegistrationModule` |
| Tab names | PascalCase constants | `CustomTabNames.Hero`, `CustomTabNames.SEO` |
| Content type GUIDs | Lowercase with hyphens, unique and stable | `"a1b2c3d4-e5f6-7890-abcd-ef1234567890"` |
| SaaS content type keys | PascalCase, matching the type name | `"BlogPostPage"`, `"HeroBlock"` |
| SaaS property keys | camelCase | `"title"`, `"heroImage"`, `"mainContentArea"` |
