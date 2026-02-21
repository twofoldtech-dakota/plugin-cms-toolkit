# Umbraco API Reference

This document covers the Content Delivery API, Management API, and common service interfaces for Umbraco v14-17.

## Table of Contents

- [Content Delivery API](#content-delivery-api)
- [Management API](#management-api)
- [Common Service Interfaces](#common-service-interfaces)

---

## Content Delivery API

The Content Delivery API is a built-in, read-only REST API for serving published content and media to front-end applications.

### Base URL and Configuration

The API base path is:

```
/umbraco/delivery/api/v2
```

Enable and configure in `appsettings.json`:

```json
{
  "Umbraco": {
    "CMS": {
      "DeliveryApi": {
        "Enabled": true,
        "PublicAccess": true,
        "ApiKey": "",
        "DisallowedContentTypeAliases": [],
        "RichTextOutputAsJson": false,
        "Media": {
          "Enabled": true
        },
        "MemberAuthorization": {
          "AuthorizationCodeFlow": {
            "Enabled": false,
            "LoginRedirectUrls": [],
            "LogoutRedirectUrls": []
          }
        },
        "OutputCache": {
          "Enabled": true,
          "ContentDuration": "00:05:00",
          "MediaDuration": "00:05:00"
        }
      }
    }
  }
}
```

Configuration options:

| Setting | Type | Description |
|---------|------|-------------|
| `Enabled` | bool | Enables the Content Delivery API |
| `PublicAccess` | bool | When `true`, the API is accessible without authentication. When `false`, requires an API key. |
| `ApiKey` | string | Optional API key for accessing the API when `PublicAccess` is `false` |
| `DisallowedContentTypeAliases` | string[] | Content type aliases to exclude from API output |
| `RichTextOutputAsJson` | bool | When `true`, Rich Text Editor output is returned as a JSON structure instead of HTML |
| `Media.Enabled` | bool | Enables the media delivery endpoint |
| `OutputCache.Enabled` | bool | Enables output caching for API responses |
| `OutputCache.ContentDuration` | TimeSpan | Cache duration for content responses |
| `OutputCache.MediaDuration` | TimeSpan | Cache duration for media responses |

### Endpoints

#### Content Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/umbraco/delivery/api/v2/content` | List published content items (paginated) |
| GET | `/umbraco/delivery/api/v2/content/item/{path}` | Fetch a single content item by its route path |
| GET | `/umbraco/delivery/api/v2/content/item/{id}` | Fetch a single content item by its GUID |
| GET | `/umbraco/delivery/api/v2/content/items` | Fetch multiple content items by IDs (query param `id`) |

#### Media Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/umbraco/delivery/api/v2/media` | List published media items (paginated) |
| GET | `/umbraco/delivery/api/v2/media/item/{id}` | Fetch a single media item by its GUID |
| GET | `/umbraco/delivery/api/v2/media/items` | Fetch multiple media items by IDs (query param `id`) |

### Query Parameters

All list endpoints support these query parameters:

| Parameter | Type | Description | Example |
|-----------|------|-------------|---------|
| `filter` | string | Filter results. Multiple filters can be combined. | `filter=contentType:articlePage` |
| `sort` | string | Sort results by a field and direction. | `sort=createDate:desc` |
| `skip` | integer | Number of items to skip (for pagination). | `skip=10` |
| `take` | integer | Number of items to return (max varies by config, default 10). | `take=25` |
| `expand` | string | Expand related content/media properties inline. | `expand=property:heroImage` |
| `fields` | string | Select specific properties to return. | `fields=properties[pageTitle,heroImage]` |
| `fetch` | string | Fetch content at a specific tree location. | `fetch=children:/blog` |

#### Filter Syntax

Filters narrow down content results:

| Filter | Description | Example |
|--------|-------------|---------|
| `contentType:{alias}` | Filter by document type alias | `filter=contentType:articlePage` |
| `name:{value}` | Filter by content name | `filter=name:My Article` |
| `createDate>{date}` | Created after date | `filter=createDate>2025-01-01` |
| `updateDate>{date}` | Updated after date | `filter=updateDate>2025-06-01` |

Multiple filters are combined with AND logic. Repeat the `filter` parameter for each condition:

```
?filter=contentType:articlePage&filter=createDate>2025-01-01
```

#### Sort Syntax

| Sort Field | Description | Example |
|------------|-------------|---------|
| `createDate:asc` | Sort by creation date ascending | `sort=createDate:asc` |
| `createDate:desc` | Sort by creation date descending | `sort=createDate:desc` |
| `updateDate:asc` | Sort by update date ascending | `sort=updateDate:asc` |
| `updateDate:desc` | Sort by update date descending | `sort=updateDate:desc` |
| `name:asc` | Sort by name ascending | `sort=name:asc` |
| `name:desc` | Sort by name descending | `sort=name:desc` |
| `sortOrder:asc` | Sort by the editor sort order | `sort=sortOrder:asc` |
| `level:asc` | Sort by tree level | `sort=level:asc` |

Multiple sorts are comma-separated:

```
?sort=createDate:desc,name:asc
```

#### Expand Syntax

Expansion replaces content/media references with their full object representation:

| Expand | Description | Example |
|--------|-------------|---------|
| `property:{alias}` | Expand a specific property | `expand=property:heroImage` |
| `property:{alias1},property:{alias2}` | Expand multiple properties | `expand=property:heroImage,property:author` |
| `all` | Expand all expandable properties | `expand=all` |

#### Fetch Syntax

The `fetch` parameter retrieves content relative to a tree position:

| Fetch | Description | Example |
|-------|-------------|---------|
| `ancestors:{path}` | Ancestors of the content at path | `fetch=ancestors:/blog/my-article` |
| `children:{path}` | Children of the content at path | `fetch=children:/blog` |
| `descendants:{path}` | Descendants of the content at path | `fetch=descendants:/blog` |

### Content Type Filtering

To exclude sensitive content types from the Delivery API, add their aliases to the disallow list:

```json
{
  "Umbraco": {
    "CMS": {
      "DeliveryApi": {
        "DisallowedContentTypeAliases": [
          "adminSettings",
          "internalPage"
        ]
      }
    }
  }
}
```

### Response Structure

#### Content List Response

```json
{
  "total": 42,
  "items": [
    {
      "name": "My Article",
      "createDate": "2025-03-15T10:30:00Z",
      "updateDate": "2025-03-16T14:22:00Z",
      "route": {
        "path": "/blog/my-article",
        "startItem": {
          "id": "a1b2c3d4-...",
          "path": "home"
        }
      },
      "id": "e5f6g7h8-...",
      "contentType": "articlePage",
      "properties": {
        "pageTitle": "My Article Title",
        "bodyContent": {
          "markup": "<p>Article body content...</p>",
          "blocks": []
        },
        "heroImage": {
          "mediaType": "Image",
          "url": "/media/1234/hero.jpg",
          "name": "Hero Image",
          "properties": {
            "altText": "Description of the image"
          }
        }
      },
      "cultures": {}
    }
  ]
}
```

#### Single Content Item Response

The same structure as an individual item from the list response, returned directly (not wrapped in `items` array).

#### Media List Response

```json
{
  "total": 100,
  "items": [
    {
      "id": "m1e2d3i4-...",
      "name": "hero-banner.jpg",
      "mediaType": "Image",
      "url": "/media/1234/hero-banner.jpg",
      "extension": "jpg",
      "width": 1920,
      "height": 1080,
      "bytes": 245678,
      "properties": {
        "altText": "A scenic mountain landscape"
      }
    }
  ]
}
```

### Request Headers

| Header | Required | Description |
|--------|----------|-------------|
| `Api-Key` | Conditional | Required when `PublicAccess` is `false` |
| `Accept-Language` | Optional | Request content in a specific culture (e.g., `en-US`, `da-DK`) |
| `Preview` | Optional | Set to `true` with appropriate auth to access draft content |
| `Start-Item` | Optional | Specify the root content node for multi-site setups |

---

## Management API

The Management API (v14+) is the backend for the Bellissima backoffice. It provides full CRUD operations for content, media, document types, and all other Umbraco entities.

### Authentication

The Management API uses OAuth2 for authentication.

#### API Key Authentication

For server-to-server communication, configure an API key:

```json
{
  "Umbraco": {
    "CMS": {
      "Management": {
        "Api": {
          "Enabled": true
        }
      }
    }
  }
}
```

API keys are managed through the backoffice under Users > API Keys. Include the key in requests:

```
Authorization: Bearer {api-key}
```

#### OAuth2 Authorization Code Flow

For interactive applications, use the Authorization Code flow:

1. Redirect to `/umbraco/management/api/v1/security/authorize`
2. User authenticates in the backoffice
3. Exchange the authorization code for an access token at `/umbraco/management/api/v1/security/token`
4. Use the access token in API requests

### Base URL

```
/umbraco/management/api/v1
```

### Document Type Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/umbraco/management/api/v1/document-type` | List all document types |
| GET | `/umbraco/management/api/v1/document-type/{id}` | Get a specific document type |
| POST | `/umbraco/management/api/v1/document-type` | Create a new document type |
| PUT | `/umbraco/management/api/v1/document-type/{id}` | Update a document type |
| DELETE | `/umbraco/management/api/v1/document-type/{id}` | Delete a document type |

### Content Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/umbraco/management/api/v1/document` | List content documents |
| GET | `/umbraco/management/api/v1/document/{id}` | Get a specific content document |
| POST | `/umbraco/management/api/v1/document` | Create a new content document |
| PUT | `/umbraco/management/api/v1/document/{id}` | Update a content document |
| DELETE | `/umbraco/management/api/v1/document/{id}` | Delete/trash a content document |
| PUT | `/umbraco/management/api/v1/document/{id}/publish` | Publish a content document |
| PUT | `/umbraco/management/api/v1/document/{id}/unpublish` | Unpublish a content document |
| POST | `/umbraco/management/api/v1/document/{id}/copy` | Copy a content document |
| PUT | `/umbraco/management/api/v1/document/{id}/move` | Move a content document |
| PUT | `/umbraco/management/api/v1/document/{id}/sort` | Sort child content documents |

### Media Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/umbraco/management/api/v1/media` | List media items |
| GET | `/umbraco/management/api/v1/media/{id}` | Get a specific media item |
| POST | `/umbraco/management/api/v1/media` | Upload/create a media item |
| PUT | `/umbraco/management/api/v1/media/{id}` | Update a media item |
| DELETE | `/umbraco/management/api/v1/media/{id}` | Delete a media item |
| PUT | `/umbraco/management/api/v1/media/{id}/move` | Move a media item |

### CRUD Operations Examples

#### Create Content

```http
POST /umbraco/management/api/v1/document
Content-Type: application/json
Authorization: Bearer {token}

{
  "contentType": {
    "id": "document-type-guid-here"
  },
  "parent": {
    "id": "parent-node-guid-here"
  },
  "variants": [
    {
      "culture": null,
      "segment": null,
      "name": "My New Article",
      "properties": [
        {
          "alias": "pageTitle",
          "value": "My New Article Title"
        },
        {
          "alias": "bodyContent",
          "value": {
            "markup": "<p>Article content goes here.</p>",
            "blocks": []
          }
        }
      ]
    }
  ]
}
```

#### Update Content

```http
PUT /umbraco/management/api/v1/document/{id}
Content-Type: application/json
Authorization: Bearer {token}

{
  "variants": [
    {
      "culture": null,
      "segment": null,
      "name": "Updated Article Title",
      "properties": [
        {
          "alias": "pageTitle",
          "value": "Updated Title"
        }
      ]
    }
  ]
}
```

#### Publish Content

```http
PUT /umbraco/management/api/v1/document/{id}/publish
Content-Type: application/json
Authorization: Bearer {token}

{
  "publishSchedules": [
    {
      "culture": null
    }
  ]
}
```

#### Upload Media

```http
POST /umbraco/management/api/v1/media
Content-Type: multipart/form-data
Authorization: Bearer {token}

--boundary
Content-Disposition: form-data; name="Id"

new-media-guid-here
--boundary
Content-Disposition: form-data; name="ContentType.Id"

image-media-type-guid
--boundary
Content-Disposition: form-data; name="Parent.Id"

parent-folder-guid
--boundary
Content-Disposition: form-data; name="File"; filename="photo.jpg"
Content-Type: image/jpeg

{binary file data}
--boundary--
```

---

## Common Service Interfaces

These are the key service interfaces available through dependency injection in Umbraco. Inject them into controllers, components, or custom services.

### IContentService

Full CRUD operations on content at the database level. Works with unpublished and published content.

```csharp
using Umbraco.Cms.Core.Services;

public class ContentManager
{
    private readonly IContentService _contentService;

    public ContentManager(IContentService contentService)
    {
        _contentService = contentService;
    }

    // Create content
    public IContent CreateArticle(string name, int parentId)
    {
        var content = _contentService.Create(name, parentId, "articlePage");
        content.SetValue("pageTitle", name);
        _contentService.Save(content);
        return content;
    }

    // Get content by ID
    public IContent? GetById(int id) => _contentService.GetById(id);

    // Get content by GUID
    public IContent? GetByKey(Guid key) => _contentService.GetById(key);

    // Update content
    public void UpdateTitle(int contentId, string newTitle)
    {
        var content = _contentService.GetById(contentId);
        if (content == null) return;

        content.SetValue("pageTitle", newTitle);
        _contentService.Save(content);
    }

    // Publish content
    public bool Publish(int contentId, int userId = -1)
    {
        var content = _contentService.GetById(contentId);
        if (content == null) return false;

        var result = _contentService.SaveAndPublish(content, userId: userId);
        return result.Success;
    }

    // Unpublish content
    public bool Unpublish(int contentId, int userId = -1)
    {
        var content = _contentService.GetById(contentId);
        if (content == null) return false;

        var result = _contentService.Unpublish(content, userId: userId);
        return result.Success;
    }

    // Delete content (moves to recycle bin)
    public void Delete(int contentId, int userId = -1)
    {
        var content = _contentService.GetById(contentId);
        if (content == null) return;

        _contentService.MoveToRecycleBin(content, userId);
    }

    // Get children
    public IEnumerable<IContent> GetChildren(int parentId)
    {
        long totalRecords;
        return _contentService.GetPagedChildren(parentId, 0, 100, out totalRecords);
    }

    // Copy content
    public IContent? Copy(int contentId, int targetParentId, int userId = -1)
    {
        var content = _contentService.GetById(contentId);
        if (content == null) return null;

        return _contentService.Copy(content, targetParentId, false, userId);
    }
}
```

### IMediaService

CRUD operations for media items.

```csharp
using Umbraco.Cms.Core.Services;
using Umbraco.Cms.Core.Models;

public class MediaManager
{
    private readonly IMediaService _mediaService;

    public MediaManager(IMediaService mediaService)
    {
        _mediaService = mediaService;
    }

    // Create a media folder
    public IMedia CreateFolder(string name, int parentId = -1)
    {
        var folder = _mediaService.CreateMedia(name, parentId, "Folder");
        _mediaService.Save(folder);
        return folder;
    }

    // Get media by ID
    public IMedia? GetById(int id) => _mediaService.GetById(id);

    // Get media by GUID
    public IMedia? GetByKey(Guid key) => _mediaService.GetById(key);

    // Get children of a media folder
    public IEnumerable<IMedia> GetChildren(int parentId)
    {
        long totalRecords;
        return _mediaService.GetPagedChildren(parentId, 0, 100, out totalRecords);
    }

    // Delete media (moves to recycle bin)
    public void Delete(int mediaId, int userId = -1)
    {
        var media = _mediaService.GetById(mediaId);
        if (media == null) return;

        _mediaService.MoveToRecycleBin(media, userId);
    }

    // Move media
    public void Move(int mediaId, int targetParentId, int userId = -1)
    {
        var media = _mediaService.GetById(mediaId);
        if (media == null) return;

        _mediaService.Move(media, targetParentId, userId);
    }
}
```

### IContentTypeService

Manage document types (content types) programmatically.

```csharp
using Umbraco.Cms.Core.Services;
using Umbraco.Cms.Core.Models;

public class ContentTypeManager
{
    private readonly IContentTypeService _contentTypeService;

    public ContentTypeManager(IContentTypeService contentTypeService)
    {
        _contentTypeService = contentTypeService;
    }

    // Get a content type by alias
    public IContentType? GetByAlias(string alias)
    {
        return _contentTypeService.Get(alias);
    }

    // Get a content type by ID
    public IContentType? GetById(int id)
    {
        return _contentTypeService.Get(id);
    }

    // Get all content types
    public IEnumerable<IContentType> GetAll()
    {
        return _contentTypeService.GetAll();
    }

    // Check if a content type exists
    public bool Exists(string alias)
    {
        return _contentTypeService.Get(alias) != null;
    }

    // Get all compositions for a content type
    public IEnumerable<IContentTypeComposition> GetCompositions(string alias)
    {
        var contentType = _contentTypeService.Get(alias);
        return contentType?.ContentTypeComposition ?? Enumerable.Empty<IContentTypeComposition>();
    }

    // Save a content type
    public void Save(IContentType contentType)
    {
        _contentTypeService.Save(contentType);
    }

    // Delete a content type
    public void Delete(string alias, int userId = -1)
    {
        var contentType = _contentTypeService.Get(alias);
        if (contentType == null) return;

        _contentTypeService.Delete(contentType, userId);
    }
}
```

### IMemberService

Manage members (front-end users) in Umbraco.

```csharp
using Umbraco.Cms.Core.Services;
using Umbraco.Cms.Core.Models;

public class MemberManager
{
    private readonly IMemberService _memberService;

    public MemberManager(IMemberService memberService)
    {
        _memberService = memberService;
    }

    // Create a member
    public IMember CreateMember(string email, string name, string memberTypeAlias)
    {
        var member = _memberService.CreateMember(
            email, email, name, memberTypeAlias);
        _memberService.Save(member);
        return member;
    }

    // Get member by email
    public IMember? GetByEmail(string email)
    {
        return _memberService.GetByEmail(email);
    }

    // Get member by ID
    public IMember? GetById(int id)
    {
        return _memberService.GetById(id);
    }

    // Get member by GUID
    public IMember? GetByKey(Guid key)
    {
        return _memberService.GetByKey(key);
    }

    // Update member properties
    public void UpdateMember(int memberId, string name, string email)
    {
        var member = _memberService.GetById(memberId);
        if (member == null) return;

        member.Name = name;
        member.Email = email;
        member.Username = email;
        _memberService.Save(member);
    }

    // Assign member to a group
    public void AssignToGroup(string memberUsername, string groupName)
    {
        _memberService.AssignRole(memberUsername, groupName);
    }

    // Remove member from a group
    public void RemoveFromGroup(string memberUsername, string groupName)
    {
        _memberService.DissociateRole(memberUsername, groupName);
    }

    // Get members by group
    public IEnumerable<IMember> GetByGroup(string groupName)
    {
        long totalRecords;
        return _memberService.GetMembersByGroup(groupName, 0, 100, out totalRecords);
    }

    // Delete member
    public void Delete(int memberId)
    {
        var member = _memberService.GetById(memberId);
        if (member == null) return;

        _memberService.Delete(member);
    }
}
```

### IPublishedContentQuery

Query the published content cache. This is the primary way to find content in views and controllers. It operates on the in-memory content cache and is very fast.

```csharp
using Umbraco.Cms.Core.Models.PublishedContent;

public class PublishedContentQueryExamples
{
    private readonly IPublishedContentQuery _contentQuery;

    public PublishedContentQueryExamples(IPublishedContentQuery contentQuery)
    {
        _contentQuery = contentQuery;
    }

    // Search content by XPath
    public IEnumerable<IPublishedContent> GetAllArticles()
    {
        return _contentQuery.ContentAtXPath("//articlePage");
    }

    // Search published content using Examine (full-text search)
    public IEnumerable<PublishedSearchResult> Search(string searchTerm)
    {
        return _contentQuery.Search(searchTerm);
    }

    // Search with specific fields
    public IEnumerable<PublishedSearchResult> SearchInFields(
        string searchTerm, params string[] fields)
    {
        return _contentQuery.Search(searchTerm, culture: null, searcherName: null);
    }

    // Get content at root
    public IEnumerable<IPublishedContent> GetRootContent()
    {
        return _contentQuery.ContentAtRoot();
    }

    // Get a single content item by GUID
    public IPublishedContent? GetById(Guid id)
    {
        return _contentQuery.Content(id);
    }

    // Get a single content item by integer ID
    public IPublishedContent? GetById(int id)
    {
        return _contentQuery.Content(id);
    }
}
```

### IUmbracoContextAccessor

Access the current Umbraco context, which provides the published content cache, media cache, and other per-request Umbraco state.

```csharp
using Umbraco.Cms.Core.Web;
using Umbraco.Cms.Core.Models.PublishedContent;

public class UmbracoContextExample
{
    private readonly IUmbracoContextAccessor _umbracoContextAccessor;

    public UmbracoContextExample(IUmbracoContextAccessor umbracoContextAccessor)
    {
        _umbracoContextAccessor = umbracoContextAccessor;
    }

    // Access the published content cache
    public IPublishedContent? GetContentByRoute(string route)
    {
        if (!_umbracoContextAccessor.TryGetUmbracoContext(out var umbracoContext))
            return null;

        var cache = umbracoContext.Content;
        return cache?.GetByRoute(route);
    }

    // Access the published media cache
    public IPublishedContent? GetMediaById(Guid id)
    {
        if (!_umbracoContextAccessor.TryGetUmbracoContext(out var umbracoContext))
            return null;

        var mediaCache = umbracoContext.Media;
        return mediaCache?.GetById(id);
    }

    // Get the content cache for querying
    public IPublishedContentCache? GetContentCache()
    {
        if (!_umbracoContextAccessor.TryGetUmbracoContext(out var umbracoContext))
            return null;

        return umbracoContext.Content;
    }

    // Check if in preview mode
    public bool IsPreview()
    {
        if (!_umbracoContextAccessor.TryGetUmbracoContext(out var umbracoContext))
            return false;

        return umbracoContext.InPreviewMode;
    }

    // Get all content of a specific type
    public IEnumerable<IPublishedContent> GetContentByType(string contentTypeAlias)
    {
        if (!_umbracoContextAccessor.TryGetUmbracoContext(out var umbracoContext))
            return Enumerable.Empty<IPublishedContent>();

        var cache = umbracoContext.Content;
        if (cache == null)
            return Enumerable.Empty<IPublishedContent>();

        return cache.GetByContentType(
            cache.GetContentType(contentTypeAlias)!)
            ?? Enumerable.Empty<IPublishedContent>();
    }
}
```

### Additional Useful Interfaces

These interfaces are frequently used alongside the primary services:

| Interface | Purpose | Common Usage |
|-----------|---------|--------------|
| `IRelationService` | Manage relations between content items | Tracking related content, bidirectional links |
| `ILocalizationService` | Manage languages and dictionary items | Multi-language setups, translated labels |
| `IExamineManager` | Access Examine/Lucene search indexes | Custom search implementations |
| `IUserService` | Manage backoffice users | Custom user provisioning, role management |
| `IDomainService` | Manage domain/hostname assignments | Multi-site routing |
| `IFileService` | Manage templates, stylesheets, scripts | Programmatic template management |
| `INotificationService` | Manage notification subscriptions | Content workflow notifications |
| `IScopeProvider` | Database transaction scoping | Custom database operations with proper scoping |
| `IPublishedValueFallback` | Configure property value fallback behavior | Language fallback, ancestor fallback |
| `IVariationContextAccessor` | Access the current variation context | Multi-language content resolution |

### Dependency Injection Registration

All Umbraco services are registered automatically. Inject them through constructor injection:

```csharp
using Microsoft.AspNetCore.Mvc;
using Umbraco.Cms.Core.Services;
using Umbraco.Cms.Core.Web;

public class MyCustomService
{
    private readonly IContentService _contentService;
    private readonly IMediaService _mediaService;
    private readonly IContentTypeService _contentTypeService;
    private readonly IMemberService _memberService;
    private readonly IPublishedContentQuery _contentQuery;
    private readonly IUmbracoContextAccessor _umbracoContextAccessor;

    public MyCustomService(
        IContentService contentService,
        IMediaService mediaService,
        IContentTypeService contentTypeService,
        IMemberService memberService,
        IPublishedContentQuery contentQuery,
        IUmbracoContextAccessor umbracoContextAccessor)
    {
        _contentService = contentService;
        _mediaService = mediaService;
        _contentTypeService = contentTypeService;
        _memberService = memberService;
        _contentQuery = contentQuery;
        _umbracoContextAccessor = umbracoContextAccessor;
    }
}
```

Register custom services in `Program.cs` or a composer:

```csharp
// In Program.cs
builder.Services.AddScoped<MyCustomService>();

// Or using a Composer
using Umbraco.Cms.Core.Composing;

public class MyComposer : IComposer
{
    public void Compose(IUmbracoBuilder builder)
    {
        builder.Services.AddScoped<MyCustomService>();
        builder.Services.AddScoped<ArticleService>();
        builder.Services.AddScoped<ContentManager>();
    }
}
```

### Important Notes on Service Usage

1. **IContentService vs IPublishedContentQuery**: Use `IContentService` for write operations and administrative tasks. Use `IPublishedContentQuery` (or `IPublishedContentCache` via `IUmbracoContextAccessor`) for read operations in front-end code. The published cache is significantly faster because it reads from an in-memory cache.

2. **UmbracoContext availability**: `IUmbracoContextAccessor` requires an active HTTP request context. It is not available in background tasks, hosted services, or notification handlers that run outside a request. Use `IUmbracoContextFactory` to create a context in those scenarios:

   ```csharp
   using Umbraco.Cms.Core.Web;

   public class BackgroundTaskService
   {
       private readonly IUmbracoContextFactory _umbracoContextFactory;

       public BackgroundTaskService(IUmbracoContextFactory umbracoContextFactory)
       {
           _umbracoContextFactory = umbracoContextFactory;
       }

       public void DoWork()
       {
           using var contextReference = _umbracoContextFactory.EnsureUmbracoContext();
           var cache = contextReference.UmbracoContext.Content;
           // Now you can query the published cache
       }
   }
   ```

3. **Scoping**: When performing multiple write operations that should be atomic, use `IScopeProvider`:

   ```csharp
   using Umbraco.Cms.Infrastructure.Scoping;

   public class AtomicOperations
   {
       private readonly IScopeProvider _scopeProvider;
       private readonly IContentService _contentService;

       public AtomicOperations(
           IScopeProvider scopeProvider,
           IContentService contentService)
       {
           _scopeProvider = scopeProvider;
           _contentService = contentService;
       }

       public void CreateMultipleItems(IEnumerable<string> names, int parentId)
       {
           using var scope = _scopeProvider.CreateScope();

           foreach (var name in names)
           {
               var content = _contentService.Create(name, parentId, "articlePage");
               content.SetValue("pageTitle", name);
               _contentService.Save(content);
           }

           scope.Complete(); // Commit all changes
       }
   }
   ```
