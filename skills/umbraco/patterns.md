# Umbraco Code Patterns Reference

This document provides concrete code patterns for Umbraco v14-17 development. All examples target the current LTS conventions and can be adapted to specific version requirements.

## Table of Contents

- [Document Type Creation (Code-First)](#document-type-creation-code-first)
- [Razor View Patterns](#razor-view-patterns)
- [Controller Patterns](#controller-patterns)
- [Content Delivery API Usage Patterns](#content-delivery-api-usage-patterns)
- [ModelsBuilder](#modelsbuilder)
- [Naming Conventions](#naming-conventions)

---

## Document Type Creation (Code-First)

### Basic Document Type

Create document types programmatically using a migration or component. This approach is useful for solution-managed schemas.

```csharp
using Umbraco.Cms.Core;
using Umbraco.Cms.Core.Models;
using Umbraco.Cms.Core.Services;
using Umbraco.Cms.Core.Strings;

public class ArticlePageDocumentType
{
    private readonly IContentTypeService _contentTypeService;
    private readonly IShortStringHelper _shortStringHelper;
    private readonly IDataTypeService _dataTypeService;

    public ArticlePageDocumentType(
        IContentTypeService contentTypeService,
        IShortStringHelper shortStringHelper,
        IDataTypeService dataTypeService)
    {
        _contentTypeService = contentTypeService;
        _shortStringHelper = shortStringHelper;
        _dataTypeService = dataTypeService;
    }

    public void Create()
    {
        if (_contentTypeService.Get("articlePage") != null)
            return;

        var contentType = new ContentType(_shortStringHelper, -1)
        {
            Alias = "articlePage",
            Name = "Article Page",
            Description = "A standard article or blog post page.",
            Icon = "icon-document",
            AllowedAsRoot = false,
            IsContainer = false,
            IsElement = false
        };

        // Add a property group (tab)
        contentType.AddPropertyGroup("content", "Content");
        contentType.AddPropertyGroup("settings", "Settings");

        // Add properties to the Content tab
        var textboxDefinition = _dataTypeService.GetByEditorAlias(
            Constants.PropertyEditors.Aliases.TextBox)?.FirstOrDefault();
        var richTextDefinition = _dataTypeService.GetByEditorAlias(
            Constants.PropertyEditors.Aliases.TinyMce)?.FirstOrDefault();
        var mediaPickerDefinition = _dataTypeService.GetByEditorAlias(
            Constants.PropertyEditors.Aliases.MediaPicker3)?.FirstOrDefault();

        if (textboxDefinition != null)
        {
            contentType.AddPropertyType(new PropertyType(
                _shortStringHelper, textboxDefinition)
            {
                Alias = "pageTitle",
                Name = "Page Title",
                Description = "The main heading displayed on the page.",
                Mandatory = true,
                ValidationRegExp = null,
                SortOrder = 0
            }, "content", "Content");
        }

        if (richTextDefinition != null)
        {
            contentType.AddPropertyType(new PropertyType(
                _shortStringHelper, richTextDefinition)
            {
                Alias = "bodyContent",
                Name = "Body Content",
                Description = "The main body content of the article.",
                Mandatory = false,
                SortOrder = 1
            }, "content", "Content");
        }

        if (mediaPickerDefinition != null)
        {
            contentType.AddPropertyType(new PropertyType(
                _shortStringHelper, mediaPickerDefinition)
            {
                Alias = "heroImage",
                Name = "Hero Image",
                Description = "The hero/banner image for this article.",
                Mandatory = false,
                SortOrder = 2
            }, "content", "Content");
        }

        // Set allowed templates
        contentType.AllowedTemplates = new[]
        {
            new ContentTypeSort(new Lazy<int>(() => -1), 0, "articlePage")
        };

        _contentTypeService.Save(contentType);
    }
}
```

### Compositions

Create a reusable composition for SEO fields that can be added to any document type.

```csharp
public class SeoCompositionCreator
{
    private readonly IContentTypeService _contentTypeService;
    private readonly IShortStringHelper _shortStringHelper;
    private readonly IDataTypeService _dataTypeService;

    public SeoCompositionCreator(
        IContentTypeService contentTypeService,
        IShortStringHelper shortStringHelper,
        IDataTypeService dataTypeService)
    {
        _contentTypeService = contentTypeService;
        _shortStringHelper = shortStringHelper;
        _dataTypeService = dataTypeService;
    }

    public IContentType CreateSeoComposition()
    {
        var existing = _contentTypeService.Get("seoComposition");
        if (existing != null)
            return existing;

        var composition = new ContentType(_shortStringHelper, -1)
        {
            Alias = "seoComposition",
            Name = "SEO Composition",
            Description = "Reusable SEO fields for any page type.",
            Icon = "icon-search",
            IsElement = false,
            AllowedAsRoot = false
        };

        composition.AddPropertyGroup("seo", "SEO");

        var textboxDef = _dataTypeService.GetByEditorAlias(
            Constants.PropertyEditors.Aliases.TextBox)?.FirstOrDefault();
        var textareaDef = _dataTypeService.GetByEditorAlias(
            Constants.PropertyEditors.Aliases.TextArea)?.FirstOrDefault();
        var mediaPickerDef = _dataTypeService.GetByEditorAlias(
            Constants.PropertyEditors.Aliases.MediaPicker3)?.FirstOrDefault();

        if (textboxDef != null)
        {
            composition.AddPropertyType(new PropertyType(
                _shortStringHelper, textboxDef)
            {
                Alias = "metaTitle",
                Name = "Meta Title",
                Description = "SEO title. Keep under 60 characters.",
                Mandatory = false,
                SortOrder = 0
            }, "seo", "SEO");
        }

        if (textareaDef != null)
        {
            composition.AddPropertyType(new PropertyType(
                _shortStringHelper, textareaDef)
            {
                Alias = "metaDescription",
                Name = "Meta Description",
                Description = "SEO description. Keep under 160 characters.",
                Mandatory = false,
                SortOrder = 1
            }, "seo", "SEO");
        }

        if (mediaPickerDef != null)
        {
            composition.AddPropertyType(new PropertyType(
                _shortStringHelper, mediaPickerDef)
            {
                Alias = "openGraphImage",
                Name = "Open Graph Image",
                Description = "Image displayed when shared on social media.",
                Mandatory = false,
                SortOrder = 2
            }, "seo", "SEO");
        }

        _contentTypeService.Save(composition);
        return composition;
    }

    public void ApplyToDocumentType(string documentTypeAlias)
    {
        var seoComposition = _contentTypeService.Get("seoComposition");
        var target = _contentTypeService.Get(documentTypeAlias);

        if (seoComposition == null || target == null)
            return;

        var compositions = target.ContentTypeComposition.ToList();
        if (!compositions.Any(c => c.Alias == "seoComposition"))
        {
            target.AddContentType(seoComposition);
            _contentTypeService.Save(target);
        }
    }
}
```

### Element Types

Create an element type for use in Block List or Block Grid editors.

```csharp
public IContentType CreateHeroBlockElementType()
{
    var existing = _contentTypeService.Get("heroBlock");
    if (existing != null)
        return existing;

    var elementType = new ContentType(_shortStringHelper, -1)
    {
        Alias = "heroBlock",
        Name = "Hero Block",
        Description = "A hero banner block with heading, text, image, and CTA.",
        Icon = "icon-picture",
        IsElement = true  // This makes it an Element Type
    };

    elementType.AddPropertyGroup("content", "Content");

    var textboxDef = _dataTypeService.GetByEditorAlias(
        Constants.PropertyEditors.Aliases.TextBox)?.FirstOrDefault();
    var textareaDef = _dataTypeService.GetByEditorAlias(
        Constants.PropertyEditors.Aliases.TextArea)?.FirstOrDefault();
    var mediaPickerDef = _dataTypeService.GetByEditorAlias(
        Constants.PropertyEditors.Aliases.MediaPicker3)?.FirstOrDefault();
    var multiUrlPickerDef = _dataTypeService.GetByEditorAlias(
        Constants.PropertyEditors.Aliases.MultiUrlPicker)?.FirstOrDefault();

    if (textboxDef != null)
    {
        elementType.AddPropertyType(new PropertyType(
            _shortStringHelper, textboxDef)
        {
            Alias = "heading",
            Name = "Heading",
            Mandatory = true,
            SortOrder = 0
        }, "content", "Content");
    }

    if (textareaDef != null)
    {
        elementType.AddPropertyType(new PropertyType(
            _shortStringHelper, textareaDef)
        {
            Alias = "subheading",
            Name = "Subheading",
            Mandatory = false,
            SortOrder = 1
        }, "content", "Content");
    }

    if (mediaPickerDef != null)
    {
        elementType.AddPropertyType(new PropertyType(
            _shortStringHelper, mediaPickerDef)
        {
            Alias = "backgroundImage",
            Name = "Background Image",
            Mandatory = true,
            SortOrder = 2
        }, "content", "Content");
    }

    if (multiUrlPickerDef != null)
    {
        elementType.AddPropertyType(new PropertyType(
            _shortStringHelper, multiUrlPickerDef)
        {
            Alias = "callToAction",
            Name = "Call to Action",
            Description = "Button link for the hero banner.",
            Mandatory = false,
            SortOrder = 3
        }, "content", "Content");
    }

    _contentTypeService.Save(elementType);
    return elementType;
}
```

Create a settings element type for the same block:

```csharp
public IContentType CreateHeroBlockSettings()
{
    var existing = _contentTypeService.Get("heroBlockSettings");
    if (existing != null)
        return existing;

    var settingsType = new ContentType(_shortStringHelper, -1)
    {
        Alias = "heroBlockSettings",
        Name = "Hero Block Settings",
        Description = "Presentation settings for the hero block.",
        Icon = "icon-settings",
        IsElement = true
    };

    settingsType.AddPropertyGroup("settings", "Settings");

    var dropdownDef = _dataTypeService.GetByEditorAlias(
        Constants.PropertyEditors.Aliases.DropDownListFlexible)?.FirstOrDefault();
    var trueFalseDef = _dataTypeService.GetByEditorAlias(
        Constants.PropertyEditors.Aliases.Boolean)?.FirstOrDefault();

    if (dropdownDef != null)
    {
        settingsType.AddPropertyType(new PropertyType(
            _shortStringHelper, dropdownDef)
        {
            Alias = "overlayStyle",
            Name = "Overlay Style",
            Description = "Select the text overlay appearance.",
            SortOrder = 0
        }, "settings", "Settings");
    }

    if (trueFalseDef != null)
    {
        settingsType.AddPropertyType(new PropertyType(
            _shortStringHelper, trueFalseDef)
        {
            Alias = "fullWidth",
            Name = "Full Width",
            Description = "Display the hero at full viewport width.",
            SortOrder = 1
        }, "settings", "Settings");
    }

    _contentTypeService.Save(settingsType);
    return settingsType;
}
```

---

## Razor View Patterns

### Strongly-Typed Views with ModelsBuilder

When ModelsBuilder is enabled (the default), each document type generates a strongly-typed model class. Use these in your templates.

```cshtml
@inherits Umbraco.Cms.Web.Common.Views.UmbracoViewPage<ContentModels.ArticlePage>
@using ContentModels = Umbraco.Cms.Web.Common.PublishedModels
@{
    Layout = "Master.cshtml";
}

<article class="article-page">
    <header class="article-header">
        <h1>@Model.PageTitle</h1>

        @if (Model.HeroImage != null)
        {
            var heroImage = Model.HeroImage;
            <div class="hero-image">
                <img src="@heroImage.GetCropUrl(width: 1200, height: 600)"
                     alt="@heroImage.Name"
                     loading="eager" />
            </div>
        }

        <div class="article-meta">
            <time datetime="@Model.CreateDate.ToString("yyyy-MM-dd")">
                @Model.CreateDate.ToString("MMMM dd, yyyy")
            </time>
        </div>
    </header>

    <div class="article-body">
        @Html.Raw(Model.BodyContent?.ToString() ?? string.Empty)
    </div>
</article>
```

### Partial Views for Components

Extract reusable components into partial views.

```cshtml
@* /Views/Partials/_ArticleCard.cshtml *@
@inherits Umbraco.Cms.Web.Common.Views.UmbracoViewPage<IPublishedContent>
@using Umbraco.Cms.Core.Models.PublishedContent
@using ContentModels = Umbraco.Cms.Web.Common.PublishedModels

@{
    var article = Model as ContentModels.ArticlePage;
    if (article == null) { return; }
}

<div class="article-card">
    @if (article.HeroImage != null)
    {
        <div class="article-card__image">
            <img src="@article.HeroImage.GetCropUrl(width: 400, height: 250)"
                 alt="@article.HeroImage.Name"
                 loading="lazy" />
        </div>
    }
    <div class="article-card__content">
        <h3 class="article-card__title">
            <a href="@article.Url()">@article.PageTitle</a>
        </h3>
        @if (!string.IsNullOrWhiteSpace(article.Value<string>("metaDescription")))
        {
            <p class="article-card__excerpt">
                @article.Value<string>("metaDescription")
            </p>
        }
        <time class="article-card__date"
              datetime="@article.CreateDate.ToString("yyyy-MM-dd")">
            @article.CreateDate.ToString("MMMM dd, yyyy")
        </time>
    </div>
</div>
```

Use the partial in a parent view:

```cshtml
@foreach (var article in Model.Children<ContentModels.ArticlePage>()
    .OrderByDescending(x => x.CreateDate)
    .Take(6))
{
    await Html.RenderPartialAsync("_ArticleCard", article);
}
```

### Block List Rendering

Render a Block List property using the built-in partial view convention.

```cshtml
@* Main view rendering the Block List *@
@inherits Umbraco.Cms.Web.Common.Views.UmbracoViewPage<ContentModels.ContentPage>
@using Umbraco.Cms.Core.Models.Blocks
@using ContentModels = Umbraco.Cms.Web.Common.PublishedModels

@{
    Layout = "Master.cshtml";
    var blocks = Model.PageBlocks; // Block List property
}

@if (blocks?.Any() == true)
{
    <div class="page-blocks">
        @await Html.GetBlockListHtmlAsync(blocks)
    </div>
}
```

Create a partial view for each block type. The naming convention is `/Views/Partials/blocklist/Components/{ElementTypeAlias}.cshtml`:

```cshtml
@* /Views/Partials/blocklist/Components/HeroBlock.cshtml *@
@inherits Umbraco.Cms.Web.Common.Views.UmbracoViewPage<BlockListItem>
@using Umbraco.Cms.Core.Models.Blocks
@using ContentModels = Umbraco.Cms.Web.Common.PublishedModels

@{
    var content = (ContentModels.HeroBlock)Model.Content;
    var settings = Model.Settings as ContentModels.HeroBlockSettings;

    var fullWidthClass = settings?.FullWidth == true ? "hero--full-width" : "";
    var overlayStyle = settings?.OverlayStyle ?? "dark";
}

<section class="hero @fullWidthClass hero--overlay-@overlayStyle">
    @if (content.BackgroundImage != null)
    {
        <div class="hero__background">
            <img src="@content.BackgroundImage.GetCropUrl(width: 1920, height: 800)"
                 alt="@content.BackgroundImage.Name"
                 loading="eager" />
        </div>
    }
    <div class="hero__content">
        <h1 class="hero__heading">@content.Heading</h1>
        @if (!string.IsNullOrWhiteSpace(content.Subheading))
        {
            <p class="hero__subheading">@content.Subheading</p>
        }
        @if (content.CallToAction?.Any() == true)
        {
            var link = content.CallToAction.First();
            <a href="@link.Url" class="hero__cta btn btn-primary"
               target="@link.Target">
                @link.Name
            </a>
        }
    </div>
</section>
```

### Block Grid Rendering

Block Grid rendering follows a similar pattern but includes layout information.

```cshtml
@* Main view rendering Block Grid *@
@inherits Umbraco.Cms.Web.Common.Views.UmbracoViewPage<ContentModels.LandingPage>
@using Umbraco.Cms.Core.Models.Blocks
@using ContentModels = Umbraco.Cms.Web.Common.PublishedModels

@{
    Layout = "Master.cshtml";
    var gridContent = Model.GridContent; // Block Grid property
}

@if (gridContent?.Any() == true)
{
    @await Html.GetBlockGridHtmlAsync(gridContent)
}
```

Block Grid partials live in `/Views/Partials/blockgrid/Components/{ElementTypeAlias}.cshtml`:

```cshtml
@* /Views/Partials/blockgrid/Components/HeroBlock.cshtml *@
@inherits Umbraco.Cms.Web.Common.Views.UmbracoViewPage<BlockGridItem>
@using Umbraco.Cms.Core.Models.Blocks
@using ContentModels = Umbraco.Cms.Web.Common.PublishedModels

@{
    var content = (ContentModels.HeroBlock)Model.Content;
    var settings = Model.Settings as ContentModels.HeroBlockSettings;
}

<section class="hero" style="grid-column: span @Model.ColumnSpan;">
    <div class="hero__content">
        <h1>@content.Heading</h1>
        @if (!string.IsNullOrWhiteSpace(content.Subheading))
        {
            <p>@content.Subheading</p>
        }
    </div>

    @* Render nested areas if the block allows child blocks *@
    @await Html.GetBlockGridItemAreasHtmlAsync(Model)
</section>
```

### Navigation Patterns Using IPublishedContent

Build navigation from the content tree.

```cshtml
@* /Views/Partials/_MainNavigation.cshtml *@
@inherits Umbraco.Cms.Web.Common.Views.UmbracoViewPage
@using Umbraco.Cms.Core.Models.PublishedContent
@using Umbraco.Extensions

@{
    var root = Model.Root();
    var currentPage = Model;
    var navItems = root.Children?
        .Where(x => x.IsVisible())
        .ToList() ?? new List<IPublishedContent>();
}

<nav class="main-nav" aria-label="Main navigation">
    <ul class="main-nav__list">
        <li class="main-nav__item @(currentPage.Id == root.Id ? "is-active" : "")">
            <a href="@root.Url()">Home</a>
        </li>
        @foreach (var item in navItems)
        {
            var isActive = currentPage.Path.Contains(item.Id.ToString());
            <li class="main-nav__item @(isActive ? "is-active" : "")">
                <a href="@item.Url()">
                    @(item.Value<string>("navigationTitle") ?? item.Name)
                </a>
                @if (item.Children?.Any(c => c.IsVisible()) == true)
                {
                    <ul class="main-nav__submenu">
                        @foreach (var child in item.Children.Where(c => c.IsVisible()))
                        {
                            <li>
                                <a href="@child.Url()">
                                    @(child.Value<string>("navigationTitle") ?? child.Name)
                                </a>
                            </li>
                        }
                    </ul>
                }
            </li>
        }
    </ul>
</nav>
```

Breadcrumb pattern:

```cshtml
@* /Views/Partials/_Breadcrumbs.cshtml *@
@inherits Umbraco.Cms.Web.Common.Views.UmbracoViewPage
@using Umbraco.Extensions

@{
    var ancestors = Model.Ancestors().Reverse().ToList();
}

<nav aria-label="Breadcrumb">
    <ol class="breadcrumbs">
        @foreach (var ancestor in ancestors)
        {
            <li class="breadcrumbs__item">
                <a href="@ancestor.Url()">@ancestor.Name</a>
            </li>
        }
        <li class="breadcrumbs__item breadcrumbs__item--current" aria-current="page">
            @Model.Name
        </li>
    </ol>
</nav>
```

---

## Controller Patterns

### Custom RenderController

Override how a specific document type renders.

```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.ViewEngines;
using Microsoft.Extensions.Logging;
using Umbraco.Cms.Core.Web;
using Umbraco.Cms.Web.Common.Controllers;

// The controller name must match the document type alias + "Controller"
public class ArticlePageController : RenderController
{
    private readonly IPublishedContentQuery _contentQuery;

    public ArticlePageController(
        ILogger<ArticlePageController> logger,
        ICompositeViewEngine compositeViewEngine,
        IUmbracoContextAccessor umbracoContextAccessor,
        IPublishedContentQuery contentQuery)
        : base(logger, compositeViewEngine, umbracoContextAccessor)
    {
        _contentQuery = contentQuery;
    }

    public override IActionResult Index()
    {
        var content = CurrentPage;
        if (content == null)
            return NotFound();

        // Build a custom view model with additional data
        var relatedArticles = _contentQuery
            .ContentAtXPath("//articlePage")
            .Where(x => x.Id != content.Id)
            .OrderByDescending(x => x.CreateDate)
            .Take(3)
            .ToList();

        var viewModel = new ArticlePageViewModel(content)
        {
            RelatedArticles = relatedArticles
        };

        return CurrentTemplate(viewModel);
    }
}

// View model extending the published content model
public class ArticlePageViewModel : ContentModel
{
    public ArticlePageViewModel(IPublishedContent content) : base(content) { }

    public IReadOnlyList<IPublishedContent> RelatedArticles { get; set; }
        = Array.Empty<IPublishedContent>();
}
```

### SurfaceController with Form Handling

Handle form submissions (e.g., a contact form).

```csharp
using Microsoft.AspNetCore.Mvc;
using Umbraco.Cms.Core.Cache;
using Umbraco.Cms.Core.Logging;
using Umbraco.Cms.Core.Routing;
using Umbraco.Cms.Core.Services;
using Umbraco.Cms.Core.Web;
using Umbraco.Cms.Infrastructure.Persistence;
using Umbraco.Cms.Web.Website.Controllers;
using System.ComponentModel.DataAnnotations;

public class ContactFormController : SurfaceController
{
    private readonly ILogger<ContactFormController> _logger;

    public ContactFormController(
        IUmbracoContextAccessor umbracoContextAccessor,
        IUmbracoDatabaseFactory databaseFactory,
        ServiceContext services,
        AppCaches appCaches,
        IProfilingLogger profilingLogger,
        IPublishedUrlProvider publishedUrlProvider,
        ILogger<ContactFormController> logger)
        : base(umbracoContextAccessor, databaseFactory, services,
               appCaches, profilingLogger, publishedUrlProvider)
    {
        _logger = logger;
    }

    [HttpPost]
    [ValidateAntiForgeryToken]
    public IActionResult SubmitContactForm(ContactFormModel model)
    {
        if (!ModelState.IsValid)
        {
            TempData["FormError"] = "Please correct the errors below.";
            return CurrentUmbracoPage();
        }

        try
        {
            // Process the form (send email, save to database, etc.)
            _logger.LogInformation(
                "Contact form submitted by {Name} ({Email})",
                model.Name, model.Email);

            TempData["FormSuccess"] = "Thank you for your message!";
            return RedirectToCurrentUmbracoPage();
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error processing contact form.");
            TempData["FormError"] = "An error occurred. Please try again.";
            return CurrentUmbracoPage();
        }
    }
}

public class ContactFormModel
{
    [Required(ErrorMessage = "Name is required.")]
    [StringLength(100)]
    public string Name { get; set; } = string.Empty;

    [Required(ErrorMessage = "Email is required.")]
    [EmailAddress(ErrorMessage = "Please enter a valid email address.")]
    public string Email { get; set; } = string.Empty;

    [Required(ErrorMessage = "Message is required.")]
    [StringLength(2000)]
    public string Message { get; set; } = string.Empty;
}
```

Razor form view:

```cshtml
@using Microsoft.AspNetCore.Mvc.TagHelpers
@model ContactFormModel

@if (TempData["FormSuccess"] != null)
{
    <div class="alert alert-success">@TempData["FormSuccess"]</div>
}
@if (TempData["FormError"] != null)
{
    <div class="alert alert-danger">@TempData["FormError"]</div>
}

@using (Html.BeginUmbracoForm<ContactFormController>(
    nameof(ContactFormController.SubmitContactForm)))
{
    <div class="form-group">
        <label asp-for="Name"></label>
        <input asp-for="Name" class="form-control" />
        <span asp-validation-for="Name" class="text-danger"></span>
    </div>

    <div class="form-group">
        <label asp-for="Email"></label>
        <input asp-for="Email" type="email" class="form-control" />
        <span asp-validation-for="Email" class="text-danger"></span>
    </div>

    <div class="form-group">
        <label asp-for="Message"></label>
        <textarea asp-for="Message" class="form-control" rows="5"></textarea>
        <span asp-validation-for="Message" class="text-danger"></span>
    </div>

    <button type="submit" class="btn btn-primary">Send Message</button>
}
```

### API Controller

Create a custom API endpoint for use by front-end JavaScript or external consumers.

```csharp
using Microsoft.AspNetCore.Mvc;
using Umbraco.Cms.Core.Models.PublishedContent;
using Umbraco.Cms.Core.Web;
using Umbraco.Extensions;

[ApiController]
[Route("api/[controller]")]
public class SearchApiController : ControllerBase
{
    private readonly IUmbracoContextAccessor _umbracoContextAccessor;
    private readonly IVariationContextAccessor _variationContextAccessor;

    public SearchApiController(
        IUmbracoContextAccessor umbracoContextAccessor,
        IVariationContextAccessor variationContextAccessor)
    {
        _umbracoContextAccessor = umbracoContextAccessor;
        _variationContextAccessor = variationContextAccessor;
    }

    [HttpGet("articles")]
    public IActionResult SearchArticles(
        [FromQuery] string? query = null,
        [FromQuery] int page = 1,
        [FromQuery] int pageSize = 10)
    {
        if (!_umbracoContextAccessor.TryGetUmbracoContext(out var umbracoContext))
            return StatusCode(500, "Umbraco context unavailable.");

        var cache = umbracoContext.Content;
        if (cache == null)
            return StatusCode(500, "Content cache unavailable.");

        var root = cache.GetAtRoot().FirstOrDefault();
        if (root == null)
            return NotFound("No root content found.");

        var articles = root
            .DescendantsOfType("articlePage")
            .Where(x => x.IsVisible());

        if (!string.IsNullOrWhiteSpace(query))
        {
            articles = articles.Where(a =>
                a.Name?.Contains(query, StringComparison.OrdinalIgnoreCase) == true ||
                a.Value<string>("pageTitle")?.Contains(query,
                    StringComparison.OrdinalIgnoreCase) == true);
        }

        var totalCount = articles.Count();
        var pagedResults = articles
            .OrderByDescending(x => x.CreateDate)
            .Skip((page - 1) * pageSize)
            .Take(pageSize)
            .Select(a => new
            {
                id = a.Key,
                title = a.Value<string>("pageTitle") ?? a.Name,
                url = a.Url(),
                date = a.CreateDate.ToString("yyyy-MM-dd")
            })
            .ToList();

        return Ok(new
        {
            totalCount,
            page,
            pageSize,
            results = pagedResults
        });
    }
}
```

---

## Content Delivery API Usage Patterns

### Basic Content Fetch

Fetch all published content:

```javascript
const response = await fetch('/umbraco/delivery/api/v2/content');
const data = await response.json();

console.log(data.total);
console.log(data.items);
```

### Fetch by Content Type

```javascript
const response = await fetch(
  '/umbraco/delivery/api/v2/content?filter=contentType:articlePage&sort=createDate:desc&take=10'
);
const data = await response.json();
```

### Fetch Single Item by Route

```javascript
async function fetchByRoute(route) {
  const response = await fetch(
    `/umbraco/delivery/api/v2/content/item/${encodeURIComponent(route)}`
  );

  if (!response.ok) {
    throw new Error(`Failed to fetch content: ${response.status}`);
  }

  return response.json();
}

// Usage
const homePage = await fetchByRoute('/');
const article = await fetchByRoute('/blog/my-article');
```

### Fetch Single Item by ID

```javascript
async function fetchById(id) {
  const response = await fetch(
    `/umbraco/delivery/api/v2/content/item/${id}`
  );

  if (!response.ok) {
    throw new Error(`Failed to fetch content: ${response.status}`);
  }

  return response.json();
}
```

### Expansion Syntax

Expand related content and media properties to include their full data inline:

```javascript
// Expand a specific property
const response = await fetch(
  '/umbraco/delivery/api/v2/content?filter=contentType:articlePage&expand=property:heroImage'
);

// Expand multiple properties
const response2 = await fetch(
  '/umbraco/delivery/api/v2/content?filter=contentType:articlePage&expand=property:heroImage,property:author'
);

// Expand all expandable properties
const response3 = await fetch(
  '/umbraco/delivery/api/v2/content?filter=contentType:articlePage&expand=all'
);
```

### Filtering and Sorting

```javascript
// Filter by content type
const byType = await fetch(
  '/umbraco/delivery/api/v2/content?filter=contentType:articlePage'
);

// Filter by name
const byName = await fetch(
  '/umbraco/delivery/api/v2/content?filter=name:My Article'
);

// Combine filters
const combined = await fetch(
  '/umbraco/delivery/api/v2/content?filter=contentType:articlePage&filter=name:My Article'
);

// Sort ascending
const sortedAsc = await fetch(
  '/umbraco/delivery/api/v2/content?sort=name:asc'
);

// Sort descending by date
const sortedDesc = await fetch(
  '/umbraco/delivery/api/v2/content?sort=createDate:desc'
);

// Pagination
const paginated = await fetch(
  '/umbraco/delivery/api/v2/content?filter=contentType:articlePage&skip=10&take=10'
);
```

### Selecting Specific Fields

Reduce payload size by selecting only the fields you need:

```javascript
const response = await fetch(
  '/umbraco/delivery/api/v2/content?filter=contentType:articlePage&fields=properties[pageTitle,heroImage,metaDescription]'
);
```

### Fetch Media

```javascript
// Fetch all media
const allMedia = await fetch('/umbraco/delivery/api/v2/media');

// Fetch media by ID
const singleMedia = await fetch(
  `/umbraco/delivery/api/v2/media/item/${mediaId}`
);

// Filter media by type
const images = await fetch(
  '/umbraco/delivery/api/v2/media?filter=mediaType:Image'
);
```

### Using an API Key

When the Delivery API requires authentication:

```javascript
const response = await fetch('/umbraco/delivery/api/v2/content', {
  headers: {
    'Api-Key': 'your-api-key-here'
  }
});
```

### Server-Side Fetch (C# HttpClient)

```csharp
using System.Net.Http;
using System.Text.Json;

public class UmbracoDeliveryApiClient
{
    private readonly HttpClient _httpClient;

    public UmbracoDeliveryApiClient(HttpClient httpClient)
    {
        _httpClient = httpClient;
    }

    public async Task<JsonDocument> GetContentByTypeAsync(
        string contentType, int take = 10, int skip = 0)
    {
        var url = $"/umbraco/delivery/api/v2/content" +
                  $"?filter=contentType:{contentType}" +
                  $"&take={take}&skip={skip}" +
                  $"&sort=createDate:desc";

        var response = await _httpClient.GetAsync(url);
        response.EnsureSuccessStatusCode();

        var stream = await response.Content.ReadAsStreamAsync();
        return await JsonDocument.ParseAsync(stream);
    }
}
```

---

## ModelsBuilder

### SourceCodeManual vs InMemoryAuto Modes

Configure ModelsBuilder in `appsettings.json`:

```json
{
  "Umbraco": {
    "CMS": {
      "ModelsBuilder": {
        "ModelsMode": "SourceCodeManual"
      }
    }
  }
}
```

| Mode | Description | When to Use |
|------|-------------|-------------|
| **InMemoryAuto** | Models generated in memory at runtime, auto-refreshed | Development, quick prototyping |
| **SourceCodeManual** | Models generated as `.cs` files, committed to source control | Production, CI/CD pipelines, team environments |
| **Nothing** | ModelsBuilder disabled | When using a custom model approach |

For team and production environments, prefer **SourceCodeManual**. This ensures models are version-controlled, visible in code reviews, and available at compile time.

### Generating Models (SourceCodeManual)

Generate models from the backoffice (Settings > Models Builder > Generate Models) or via the command line:

```bash
dotnet run -- models generate
```

Generated models are placed in `/umbraco/models/` by default. Configure the output directory:

```json
{
  "Umbraco": {
    "CMS": {
      "ModelsBuilder": {
        "ModelsMode": "SourceCodeManual",
        "ModelsDirectory": "~/Models/Generated"
      }
    }
  }
}
```

### Custom Properties and Extensions

Extend generated models using partial classes. Never modify generated files directly.

```csharp
// /Models/Generated/ArticlePage.generated.cs (auto-generated, do not edit)
// This file is generated by ModelsBuilder.

// /Models/ArticlePageExtensions.cs (your custom code)
using Umbraco.Cms.Core.Models.PublishedContent;

namespace Umbraco.Cms.Web.Common.PublishedModels
{
    public partial class ArticlePage
    {
        /// <summary>
        /// Returns the page title, falling back to the node name.
        /// </summary>
        public string DisplayTitle =>
            !string.IsNullOrWhiteSpace(PageTitle)
                ? PageTitle
                : Name;

        /// <summary>
        /// Returns a truncated excerpt from the body content.
        /// </summary>
        public string Excerpt(int maxLength = 200)
        {
            var text = BodyContent?.ToString() ?? string.Empty;
            // Strip HTML tags for plain text excerpt
            var plainText = System.Text.RegularExpressions.Regex.Replace(
                text, "<[^>]+>", string.Empty);

            if (plainText.Length <= maxLength)
                return plainText;

            return plainText.Substring(0, maxLength).TrimEnd() + "...";
        }

        /// <summary>
        /// Gets the SEO title, falling back to the display title.
        /// </summary>
        public string SeoTitle =>
            !string.IsNullOrWhiteSpace(this.Value<string>("metaTitle"))
                ? this.Value<string>("metaTitle")!
                : DisplayTitle;
    }
}
```

### Generated Model Usage in Controllers and Services

```csharp
using Umbraco.Cms.Core.Models.PublishedContent;
using Umbraco.Cms.Web.Common.PublishedModels;

public class ArticleService
{
    private readonly IPublishedContentQuery _contentQuery;

    public ArticleService(IPublishedContentQuery contentQuery)
    {
        _contentQuery = contentQuery;
    }

    public IEnumerable<ArticlePage> GetLatestArticles(int count = 5)
    {
        return _contentQuery
            .ContentAtXPath("//articlePage")
            .OfType<ArticlePage>()
            .OrderByDescending(x => x.CreateDate)
            .Take(count);
    }

    public IEnumerable<ArticlePage> GetArticlesByTag(string tag)
    {
        return _contentQuery
            .ContentAtXPath("//articlePage")
            .OfType<ArticlePage>()
            .Where(a => a.Value<IEnumerable<string>>("tags")
                ?.Contains(tag, StringComparer.OrdinalIgnoreCase) == true)
            .OrderByDescending(x => x.CreateDate);
    }
}
```

---

## Naming Conventions

| Item | Alias Format | Name/Label Format | Example Alias | Example Name |
|------|-------------|-------------------|---------------|--------------|
| **Document Type** | camelCase | Readable with spaces | `articlePage` | `Article Page` |
| **Element Type** | camelCase | Readable with spaces | `heroBlock` | `Hero Block` |
| **Composition** | camelCase with `Composition` suffix | Readable with `Composition` suffix | `seoComposition` | `SEO Composition` |
| **Property** | camelCase | Readable with spaces | `pageTitle` | `Page Title` |
| **Property** | camelCase | Readable with spaces | `heroImage` | `Hero Image` |
| **Property** | camelCase | Readable with spaces | `bodyContent` | `Body Content` |
| **Tab/Group** | camelCase | Readable (short, descriptive) | `content` | `Content` |
| **Tab/Group** | camelCase | Readable (short, descriptive) | `seo` | `SEO` |
| **Template** | PascalCase matching doc type | Matches document type name | `ArticlePage` | `Article Page` |
| **Partial View** | Underscore prefix or descriptive | Descriptive | `_ArticleCard.cshtml` | -- |
| **Partial View** | Descriptive | Descriptive | `_MainNavigation.cshtml` | -- |
| **Controller** | DocTypeAlias + `Controller` | -- | `ArticlePageController` | -- |
| **SurfaceController** | Descriptive + `Controller` | -- | `ContactFormController` | -- |
| **View Model** | DocTypeAlias + `ViewModel` | -- | `ArticlePageViewModel` | -- |
| **Composition File** | PascalCase | -- | `SeoComposition.cs` | -- |

### Folder Structure Conventions

```
/Views/
  Master.cshtml                          # Main layout
  ArticlePage.cshtml                     # Template matching document type
  HomePage.cshtml
  ContentPage.cshtml
  /Partials/
    _MainNavigation.cshtml               # Shared partial views
    _Breadcrumbs.cshtml
    _ArticleCard.cshtml
    /blocklist/
      /Components/
        HeroBlock.cshtml                 # Block List item partials
        CtaBlock.cshtml
        TestimonialBlock.cshtml
    /blockgrid/
      /Components/
        HeroBlock.cshtml                 # Block Grid item partials
        TwoColumnBlock.cshtml

/Controllers/
  ArticlePageController.cs               # RenderControllers
  ContactFormController.cs               # SurfaceControllers

/Models/
  ArticlePageViewModel.cs                # Custom view models
  ContactFormModel.cs                    # Form models
  /Generated/
    ArticlePage.generated.cs             # ModelsBuilder output (SourceCodeManual)
    HomePage.generated.cs
```
