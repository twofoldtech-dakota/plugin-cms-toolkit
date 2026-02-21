# Optimizely CMS API Reference

API reference for CMS SaaS REST API, Optimizely Graph (GraphQL), and CMS 12 core .NET services.

## CMS SaaS REST API

### Base URL

```
https://{instance}.cms.optimizely.com
```

Replace `{instance}` with your CMS SaaS instance identifier.

### Authentication

CMS SaaS uses OAuth2 with client credentials:

```bash
POST https://id.optimizely.com/oauth/token
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials
&client_id={client_id}
&client_secret={client_secret}
&scope=openid
```

Response:

```json
{
  "access_token": "eyJ...",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

Use the token in all subsequent requests:

```
Authorization: Bearer {access_token}
```

API keys can also be used for server-to-server calls:

```
Authorization: ApiKey {api_key}
```

### Content Types API

Base path: `/api/episerver/v3.0/contenttypes`

#### List All Content Types

```http
GET /api/episerver/v3.0/contenttypes
Authorization: Bearer {token}
```

Response:

```json
[
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
      }
    }
  }
]
```

#### Get a Specific Content Type

```http
GET /api/episerver/v3.0/contenttypes/{key}
Authorization: Bearer {token}
```

#### Create a Content Type

```http
POST /api/episerver/v3.0/contenttypes
Authorization: Bearer {token}
Content-Type: application/json

{
  "key": "TestimonialBlock",
  "baseType": "Block",
  "displayName": "Testimonial Block",
  "description": "A customer testimonial with quote, author, and photo",
  "properties": {
    "quote": {
      "type": "RichText",
      "displayName": "Quote",
      "required": true,
      "localized": true,
      "sortOrder": 10
    },
    "authorName": {
      "type": "String",
      "displayName": "Author Name",
      "required": true,
      "sortOrder": 20
    },
    "authorTitle": {
      "type": "String",
      "displayName": "Author Title",
      "sortOrder": 30
    },
    "authorPhoto": {
      "type": "ContentReference",
      "displayName": "Author Photo",
      "allowedTypes": ["Image"],
      "sortOrder": 40
    },
    "rating": {
      "type": "Integer",
      "displayName": "Rating (1-5)",
      "sortOrder": 50
    }
  }
}
```

#### Update a Content Type

```http
PUT /api/episerver/v3.0/contenttypes/{key}
Authorization: Bearer {token}
Content-Type: application/json

{
  "key": "TestimonialBlock",
  "baseType": "Block",
  "displayName": "Testimonial Block (Updated)",
  "properties": {
    "quote": {
      "type": "RichText",
      "displayName": "Quote",
      "required": true,
      "localized": true,
      "sortOrder": 10
    },
    "authorName": {
      "type": "String",
      "displayName": "Author Name",
      "required": true,
      "sortOrder": 20
    },
    "companyName": {
      "type": "String",
      "displayName": "Company Name",
      "sortOrder": 25
    }
  }
}
```

#### Delete a Content Type

```http
DELETE /api/episerver/v3.0/contenttypes/{key}
Authorization: Bearer {token}
```

Returns `204 No Content` on success. Cannot delete a content type that has existing content instances.

### Content API

Base path: `/api/episerver/v3.0/content`

#### List Content Items

```http
GET /api/episerver/v3.0/content
Authorization: Bearer {token}
```

#### Query Parameters for Content Listing

| Parameter | Type | Description |
|-----------|------|-------------|
| `contentTypes` | string | Filter by content type key (comma-separated) |
| `parentId` | integer | Filter by parent content ID |
| `language` | string | Language branch (e.g., `en`, `sv`) |
| `status` | string | `Published`, `Draft`, `PreviouslyPublished` |
| `orderBy` | string | Property to sort by (e.g., `name`, `changed`) |
| `direction` | string | `asc` or `desc` |
| `top` | integer | Number of items to return (max 100) |
| `skip` | integer | Number of items to skip for pagination |
| `expand` | string | Properties to expand (e.g., `*` for all) |

Example with filters:

```http
GET /api/episerver/v3.0/content?contentTypes=ArticlePage&language=en&status=Published&top=10&orderBy=changed&direction=desc
Authorization: Bearer {token}
```

#### Get a Specific Content Item

```http
GET /api/episerver/v3.0/content/{id}
Authorization: Bearer {token}
```

With language:

```http
GET /api/episerver/v3.0/content/{id}?language=en
Authorization: Bearer {token}
```

With expanded references:

```http
GET /api/episerver/v3.0/content/{id}?expand=heroImage,relatedArticles
Authorization: Bearer {token}
```

Response:

```json
{
  "contentLink": {
    "id": 123,
    "guidValue": "a1b2c3d4-e5f6-7890-abcd-ef1234567890"
  },
  "name": "My Article",
  "contentType": ["ArticlePage"],
  "parentLink": {
    "id": 5
  },
  "language": {
    "name": "en",
    "displayName": "English"
  },
  "existingLanguages": [
    { "name": "en", "displayName": "English" },
    { "name": "sv", "displayName": "Swedish" }
  ],
  "status": "Published",
  "changed": "2025-06-15T10:30:00Z",
  "title": {
    "value": "My Article Title"
  },
  "body": {
    "value": "<p>Article body content...</p>"
  },
  "heroImage": {
    "value": {
      "id": 456,
      "url": "https://assets.optimizely.com/.../hero.jpg"
    }
  }
}
```

#### Create Content

```http
POST /api/episerver/v3.0/content
Authorization: Bearer {token}
Content-Type: application/json

{
  "name": "New Blog Post",
  "contentType": ["BlogPostPage"],
  "parentLink": {
    "id": 5
  },
  "language": {
    "name": "en"
  },
  "status": "Draft",
  "title": {
    "value": "New Blog Post Title"
  },
  "body": {
    "value": "<p>Blog post content here.</p>"
  }
}
```

Returns `201 Created` with the new content item in the response body.

#### Update Content (Full Replace)

```http
PUT /api/episerver/v3.0/content/{id}
Authorization: Bearer {token}
Content-Type: application/json

{
  "name": "Updated Blog Post",
  "contentType": ["BlogPostPage"],
  "parentLink": {
    "id": 5
  },
  "language": {
    "name": "en"
  },
  "status": "Published",
  "title": {
    "value": "Updated Title"
  },
  "body": {
    "value": "<p>Updated body content.</p>"
  }
}
```

#### Partial Update (Patch)

```http
PATCH /api/episerver/v3.0/content/{id}
Authorization: Bearer {token}
Content-Type: application/json

{
  "title": {
    "value": "Patched Title Only"
  },
  "status": "Published"
}
```

#### Delete Content

```http
DELETE /api/episerver/v3.0/content/{id}
Authorization: Bearer {token}
```

Returns `204 No Content` on success. By default moves to trash. Add `?permanent=true` to permanently delete.

#### Move Content

```http
PUT /api/episerver/v3.0/content/{id}/move
Authorization: Bearer {token}
Content-Type: application/json

{
  "parentLink": {
    "id": 10
  }
}
```

### Media API

Media assets use the same Content API endpoints but with media-specific content types.

#### Upload Media

```http
POST /api/episerver/v3.0/content
Authorization: Bearer {token}
Content-Type: multipart/form-data

--boundary
Content-Disposition: form-data; name="file"; filename="hero.jpg"
Content-Type: image/jpeg

{binary data}
--boundary
Content-Disposition: form-data; name="metadata"
Content-Type: application/json

{
  "name": "Hero Image",
  "contentType": ["Image"],
  "parentLink": { "id": 100 },
  "altText": { "value": "A hero banner image" }
}
--boundary--
```

#### Get Media by ID

```http
GET /api/episerver/v3.0/content/{mediaId}
Authorization: Bearer {token}
```

The response includes a `url` field pointing to the asset CDN URL.

## Graph API (Optimizely Graph)

### Endpoint and Authentication

GraphQL endpoint:

```
https://cg.optimizely.com/content/v2
```

Authentication headers:

```
Authorization: Basic {base64(AppKey:Secret)}
```

Or use a single key:

```
Authorization: epi-single {singleKey}
```

### Content Queries

#### Basic Query

```graphql
query {
  ArticlePage(locale: en, limit: 10, orderBy: { changed: DESC }) {
    items {
      title
      body {
        html
      }
      heroImage {
        url
      }
      changed
      _link {
        default
      }
    }
    total
  }
}
```

#### Filtering

Operators available on most property types:

| Operator | Description | Example |
|----------|-------------|---------|
| `eq` | Equals | `{ status: { eq: "Published" } }` |
| `notEq` | Not equals | `{ category: { notEq: "Draft" } }` |
| `in` | In list | `{ category: { in: ["News", "Blog"] } }` |
| `notIn` | Not in list | `{ category: { notIn: ["Archive"] } }` |
| `lt` / `lte` | Less than (or equal) | `{ rating: { gte: 4 } }` |
| `gt` / `gte` | Greater than (or equal) | `{ publishDate: { gt: "2025-01-01" } }` |
| `like` | Wildcard match | `{ title: { like: "%optimizely%" } }` |
| `contains` | Contains string | `{ title: { contains: "guide" } }` |
| `startsWith` | Starts with | `{ title: { startsWith: "How" } }` |
| `exists` | Property has value | `{ heroImage: { exists: true } }` |

Combined filters:

```graphql
query {
  BlogPostPage(
    where: {
      _and: [
        { publishDate: { gte: "2025-01-01" } }
        { featured: { eq: true } }
        { category: { in: ["Technology", "Engineering"] } }
      ]
    }
    limit: 20
  ) {
    items {
      title
      publishDate
      category
    }
    total
  }
}
```

#### Full-Text Search

```graphql
query Search($term: String!) {
  ArticlePage(
    where: { _fulltext: { match: $term } }
    orderBy: { _ranking: SEMANTIC }
    limit: 20
  ) {
    items {
      title
      body {
        html
      }
      _score
    }
    total
  }
}
```

#### Facets

```graphql
query {
  BlogPostPage(limit: 0) {
    facets {
      category(limit: 20) {
        name
        count
      }
      authorName(limit: 10) {
        name
        count
      }
      publishDate(unit: MONTH) {
        name
        count
      }
    }
    total
  }
}
```

#### Pagination with Cursors

```graphql
query {
  ArticlePage(
    limit: 10
    cursor: "eyJzIjoiMTAifQ=="
    orderBy: { publishDate: DESC }
  ) {
    items {
      title
      publishDate
    }
    cursor
    total
  }
}
```

#### Nested Content and References

```graphql
query {
  LandingPage(locale: en) {
    items {
      title
      mainContentArea {
        __typename
        ... on HeroBlock {
          heading
          body {
            html
          }
          backgroundImage {
            url
          }
        }
        ... on TextBlock {
          heading
          body {
            html
          }
        }
        ... on CallToActionBlock {
          heading
          buttonText
          buttonLink
        }
      }
    }
  }
}
```

### Subscriptions (Webhooks)

Optimizely Graph supports webhook subscriptions for content changes:

```http
POST https://cg.optimizely.com/api/webhooks
Authorization: Basic {credentials}
Content-Type: application/json

{
  "url": "https://your-app.com/webhook/content-updated",
  "topics": ["doc/*"],
  "secret": "your-webhook-secret"
}
```

Topics:
- `doc/create` — Content created
- `doc/update` — Content updated
- `doc/delete` — Content deleted
- `doc/*` — All content changes

### Integration with CMS 12

Install the `Optimizely.ContentGraph.Cms` package to sync CMS 12 content into Graph:

```bash
dotnet add package Optimizely.ContentGraph.Cms
```

Configuration in `appsettings.json`:

```json
{
  "Optimizely": {
    "ContentGraph": {
      "GatewayAddress": "https://cg.optimizely.com",
      "AppKey": "your-app-key",
      "Secret": "your-secret",
      "SingleKey": "your-single-key",
      "AllowSendingLog": true
    }
  }
}
```

Register in `Startup.cs`:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddContentGraph(Configuration);
}
```

Content is automatically synced to Graph when published, updated, or deleted in CMS 12. You can also trigger a full sync via the scheduled job "Optimizely Graph content synchronization job."

### Integration with CMS SaaS

CMS SaaS content is automatically available in Graph. No additional configuration is required. Use the Graph API endpoint with your instance credentials.

## CMS 12 Core Services

### IContentRepository

Full CRUD service for content operations. Inject via constructor.

```csharp
using EPiServer;
using EPiServer.Core;
using EPiServer.DataAccess;
using EPiServer.Security;
```

| Method | Signature | Description |
|--------|-----------|-------------|
| `Get<T>` | `T Get<T>(ContentReference contentLink)` | Get content by reference |
| `Get<T>` | `T Get<T>(ContentReference contentLink, LanguageSelector selector)` | Get content in specific language |
| `GetChildren<T>` | `IEnumerable<T> GetChildren<T>(ContentReference contentLink)` | Get child content items |
| `GetAncestors` | `IEnumerable<IContent> GetAncestors(ContentReference contentLink)` | Get parent chain to root |
| `GetDescendents` | `IEnumerable<ContentReference> GetDescendents(ContentReference contentLink)` | Get all descendant references |
| `GetDefault<T>` | `T GetDefault<T>(ContentReference parentLink)` | Create new content instance |
| `GetDefault<T>` | `T GetDefault<T>(ContentReference parentLink, ContentType contentType)` | Create new content of specified type |
| `Save` | `ContentReference Save(IContent content, SaveAction action, AccessLevel access)` | Save content with specified action |
| `Move` | `void Move(ContentReference contentLink, ContentReference destination, AccessLevel requiredSourceAccess, AccessLevel requiredDestAccess)` | Move content to new parent |
| `Copy` | `ContentReference Copy(ContentReference source, ContentReference destination, AccessLevel requiredSourceAccess, AccessLevel requiredDestAccess, bool publishOnDestination)` | Copy content to new location |
| `Delete` | `void Delete(ContentReference contentLink, bool forceDelete)` | Delete content (forceDelete=true bypasses trash) |
| `DeleteChildren` | `void DeleteChildren(ContentReference contentLink, bool forceDelete)` | Delete all children |
| `DeleteLanguageBranch` | `void DeleteLanguageBranch(ContentReference contentLink, string languageBranch, AccessLevel access)` | Delete specific language version |

SaveAction values:

| Value | Description |
|-------|-------------|
| `SaveAction.Publish` | Save and publish immediately |
| `SaveAction.ForLater` | Save as draft |
| `SaveAction.CheckIn` | Check in for review/approval |
| `SaveAction.RequestApproval` | Submit for approval workflow |
| `SaveAction.Reject` | Reject content in approval workflow |
| `SaveAction.Schedule` | Schedule for future publish |

### IContentLoader

Read-only content loading. Prefer this over `IContentRepository` when you only need to read.

```csharp
using EPiServer;
using EPiServer.Core;
```

| Method | Signature | Description |
|--------|-----------|-------------|
| `Get<T>` | `T Get<T>(ContentReference contentLink)` | Get content by reference |
| `Get<T>` | `T Get<T>(ContentReference contentLink, LoaderOptions options)` | Get with loader options |
| `TryGet<T>` | `bool TryGet<T>(ContentReference contentLink, out T content)` | Safe get that returns false if not found |
| `GetChildren<T>` | `IEnumerable<T> GetChildren<T>(ContentReference contentLink)` | Get children of content |
| `GetChildren<T>` | `IEnumerable<T> GetChildren<T>(ContentReference contentLink, LoaderOptions options, int startIndex, int maxRows)` | Get children with paging |
| `GetAncestors` | `IEnumerable<IContent> GetAncestors(ContentReference contentLink)` | Get ancestor chain |
| `GetItems` | `IEnumerable<IContent> GetItems(IEnumerable<ContentReference> contentLinks, LoaderOptions options)` | Batch load multiple items |
| `GetBySegment` | `IContent GetBySegment(ContentReference parentLink, string urlSegment, LoaderOptions options)` | Get content by URL segment |

### IContentTypeRepository

Manage content type definitions at runtime.

```csharp
using EPiServer.DataAbstraction;
```

| Method | Signature | Description |
|--------|-----------|-------------|
| `Load` | `ContentType Load(int id)` | Load content type by ID |
| `Load` | `ContentType Load(string name)` | Load content type by name |
| `Load` | `ContentType Load(Guid guid)` | Load content type by GUID |
| `List` | `IEnumerable<ContentType> List()` | List all content types |
| `Save` | `void Save(ContentType contentType)` | Save changes to a content type |
| `Delete` | `void Delete(ContentType contentType)` | Delete a content type |

Example:

```csharp
public class ContentTypeService
{
    private readonly IContentTypeRepository _contentTypeRepository;

    public ContentTypeService(IContentTypeRepository contentTypeRepository)
    {
        _contentTypeRepository = contentTypeRepository;
    }

    public ContentType GetContentType(string name)
    {
        return _contentTypeRepository.Load(name);
    }

    public IEnumerable<ContentType> GetPageTypes()
    {
        return _contentTypeRepository.List()
            .Where(ct => typeof(PageData).IsAssignableFrom(ct.ModelType));
    }
}
```

### IContentEvents

Event hooks for content lifecycle. Subscribe in an initialization module.

```csharp
using EPiServer;
using EPiServer.Core;
```

| Event | Fires When | EventArgs Type |
|-------|-----------|----------------|
| `CreatingContent` | Before content is created | `ContentEventArgs` |
| `CreatedContent` | After content is created | `ContentEventArgs` |
| `SavingContent` | Before content is saved | `ContentEventArgs` |
| `SavedContent` | After content is saved | `ContentEventArgs` |
| `PublishingContent` | Before content is published | `ContentEventArgs` |
| `PublishedContent` | After content is published | `ContentEventArgs` |
| `DeletingContent` | Before content is deleted | `DeleteContentEventArgs` |
| `DeletedContent` | After content is deleted | `ContentEventArgs` |
| `MovingContent` | Before content is moved | `ContentEventArgs` |
| `MovedContent` | After content is moved | `ContentEventArgs` |
| `CheckingInContent` | Before content is checked in | `ContentEventArgs` |
| `CheckedInContent` | After content is checked in | `ContentEventArgs` |
| `LoadingContent` | Before content is loaded (use sparingly) | `ContentEventArgs` |
| `LoadedContent` | After content is loaded (use sparingly) | `ContentEventArgs` |

Cancelling an event:

```csharp
private void OnSavingContent(object sender, ContentEventArgs e)
{
    if (e.Content is ArticlePage article && string.IsNullOrEmpty(article.MetaDescription))
    {
        e.CancelAction = true;
        e.CancelReason = "Meta description is required for article pages.";
    }
}
```

### ILanguageBranchRepository

Manage available languages.

```csharp
using EPiServer.DataAbstraction;
```

| Method | Signature | Description |
|--------|-----------|-------------|
| `ListEnabled` | `IEnumerable<LanguageBranch> ListEnabled()` | Get all enabled languages |
| `ListAll` | `IEnumerable<LanguageBranch> ListAll()` | Get all languages (enabled and disabled) |
| `Load` | `LanguageBranch Load(string languageCode)` | Load a language branch by code |
| `Load` | `LanguageBranch Load(CultureInfo culture)` | Load a language branch by culture |
| `Save` | `void Save(LanguageBranch languageBranch)` | Save changes to a language branch |
| `Delete` | `void Delete(LanguageBranch languageBranch)` | Delete a language branch |

Example:

```csharp
public class LanguageService
{
    private readonly ILanguageBranchRepository _languageBranchRepository;

    public LanguageService(ILanguageBranchRepository languageBranchRepository)
    {
        _languageBranchRepository = languageBranchRepository;
    }

    public IEnumerable<string> GetEnabledLanguageCodes()
    {
        return _languageBranchRepository.ListEnabled()
            .Select(lb => lb.LanguageID);
    }

    public bool IsLanguageEnabled(string languageCode)
    {
        var branch = _languageBranchRepository.Load(languageCode);
        return branch?.Enabled == true;
    }
}
```

### ServiceLocator (and Why to Prefer DI)

`ServiceLocator` is a static service resolution pattern available in Optimizely. It exists for backward compatibility and for contexts where constructor injection is not available.

```csharp
using EPiServer.ServiceLocation;

// Service locator usage (avoid when possible)
var contentLoader = ServiceLocator.Current.GetInstance<IContentLoader>();
var contentRepository = ServiceLocator.Current.GetInstance<IContentRepository>();
```

**Why to prefer dependency injection instead:**

| Concern | ServiceLocator | Constructor Injection |
|---------|---------------|----------------------|
| Testability | Hard to mock; requires static setup | Easy to mock; pass in test doubles |
| Explicit dependencies | Hidden; callers do not know what the class needs | Visible in the constructor signature |
| Lifetime management | No control; always resolves from container | Container manages scoped, transient, singleton lifetimes |
| Framework alignment | Legacy pattern from EPiServer; discouraged in .NET Core | Standard ASP.NET Core pattern |
| Refactoring safety | Adding a dependency is invisible to callers | Adding a constructor parameter surfaces missing registrations at startup |

**When ServiceLocator is acceptable:**
- In content model classes (e.g., computed properties on `PageData`) where DI is not available
- In scheduled jobs on older CMS 12 versions before constructor injection was supported
- In static extension methods that need CMS services

**Preferred approach:**

```csharp
// Preferred: constructor injection
public class MyService
{
    private readonly IContentLoader _contentLoader;
    private readonly IContentRepository _contentRepository;

    public MyService(IContentLoader contentLoader, IContentRepository contentRepository)
    {
        _contentLoader = contentLoader;
        _contentRepository = contentRepository;
    }
}
```
