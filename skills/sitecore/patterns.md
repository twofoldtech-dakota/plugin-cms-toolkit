# Sitecore Code Patterns Reference

Detailed code patterns for Sitecore XM Cloud (Content SDK / Next.js), SXA, traditional MVC, and GraphQL.

## Content SDK Component Patterns (Next.js)

### Basic Component with Props Interface

Every Content SDK component receives a `ComponentRendering` and optional `layoutData`. Define a typed props interface for the datasource fields.

```tsx
import {
  Text,
  TextField,
  ComponentRendering,
} from '@sitecore-content-sdk/nextjs';

interface HeroBannerFields {
  Heading: TextField;
  Subheading: TextField;
}

interface HeroBannerProps {
  rendering: ComponentRendering;
  fields: HeroBannerFields;
}

const HeroBanner = (props: HeroBannerProps): JSX.Element => {
  return (
    <section className="hero-banner" data-component="HeroBanner">
      <h1>
        <Text field={props.fields.Heading} />
      </h1>
      <p>
        <Text field={props.fields.Subheading} />
      </p>
    </section>
  );
};

export default HeroBanner;
```

### Component with Datasource Fields (All Field Types)

Demonstrates the common field helper components for Image, Link, and RichText alongside Text.

```tsx
import {
  Text,
  TextField,
  RichText,
  RichTextField,
  Image,
  ImageField,
  Link,
  LinkField,
  ComponentRendering,
} from '@sitecore-content-sdk/nextjs';

interface ArticleCardFields {
  Title: TextField;
  Summary: RichTextField;
  Thumbnail: ImageField;
  ReadMoreLink: LinkField;
}

interface ArticleCardProps {
  rendering: ComponentRendering;
  fields: ArticleCardFields;
}

const ArticleCard = (props: ArticleCardProps): JSX.Element => {
  return (
    <article className="article-card">
      <Image field={props.fields.Thumbnail} />
      <h2>
        <Text field={props.fields.Title} />
      </h2>
      <RichText field={props.fields.Summary} />
      <Link field={props.fields.ReadMoreLink} className="read-more" />
    </article>
  );
};

export default ArticleCard;
```

### Component Using useSitecoreContext

Use `useSitecoreContext` to access page-level data such as the current route, site info, and page fields -- data that lives on the route item rather than a datasource.

```tsx
import {
  useSitecoreContext,
  Text,
  TextField,
  ImageField,
  Image,
} from '@sitecore-content-sdk/nextjs';

interface PageHeaderFields {
  PageTitle: TextField;
  PageImage: ImageField;
}

const PageHeader = (): JSX.Element | null => {
  const { sitecoreContext } = useSitecoreContext();
  const fields = sitecoreContext.route?.fields as unknown as PageHeaderFields;

  if (!fields) {
    return null;
  }

  return (
    <header className="page-header">
      <Image field={fields.PageImage} />
      <h1>
        <Text field={fields.PageTitle} />
      </h1>
      <p>Current language: {sitecoreContext.language}</p>
    </header>
  );
};

export default PageHeader;
```

### Component with Rendering Parameters

Rendering parameters provide author-configurable presentation settings. Access them via `props.rendering.params`.

```tsx
import {
  Text,
  TextField,
  RichText,
  RichTextField,
  ComponentRendering,
} from '@sitecore-content-sdk/nextjs';

interface ContentBlockFields {
  Heading: TextField;
  Body: RichTextField;
}

interface ContentBlockProps {
  rendering: ComponentRendering;
  fields: ContentBlockFields;
}

const ContentBlock = (props: ContentBlockProps): JSX.Element => {
  const { params } = props.rendering;
  const theme = params?.Theme || 'light';
  const columns = params?.Columns || '1';

  return (
    <section
      className={`content-block content-block--${theme}`}
      data-columns={columns}
    >
      <h2>
        <Text field={props.fields.Heading} />
      </h2>
      <div className={`content-block__body content-block__body--cols-${columns}`}>
        <RichText field={props.fields.Body} />
      </div>
    </section>
  );
};

export default ContentBlock;
```

### Component with Placeholder

Placeholders allow authors to nest other components inside a parent component.

```tsx
import {
  Text,
  TextField,
  Placeholder,
  ComponentRendering,
} from '@sitecore-content-sdk/nextjs';

interface TabContainerFields {
  Title: TextField;
}

interface TabContainerProps {
  rendering: ComponentRendering;
  fields: TabContainerFields;
}

const TabContainer = (props: TabContainerProps): JSX.Element => {
  return (
    <div className="tab-container">
      <h2>
        <Text field={props.fields.Title} />
      </h2>
      <div className="tab-container__content">
        <Placeholder name="tab-content" rendering={props.rendering} />
      </div>
    </div>
  );
};

export default TabContainer;
```

### Handling Missing Datasource Gracefully

Always handle the case where a datasource is not set. Display an editing message in Experience Editor / Pages, and render nothing on the public site.

```tsx
import {
  Text,
  TextField,
  useSitecoreContext,
  ComponentRendering,
} from '@sitecore-content-sdk/nextjs';

interface PromoFields {
  Headline: TextField;
}

interface PromoProps {
  rendering: ComponentRendering;
  fields: PromoFields;
}

const Promo = (props: PromoProps): JSX.Element | null => {
  const { sitecoreContext } = useSitecoreContext();
  const isEditing = sitecoreContext.pageEditing;

  if (!props.fields) {
    if (isEditing) {
      return (
        <div className="promo promo--empty">
          <p>Select a datasource item for this Promo component.</p>
        </div>
      );
    }
    return null;
  }

  return (
    <div className="promo">
      <h3>
        <Text field={props.fields.Headline} />
      </h3>
    </div>
  );
};

export default Promo;
```

### Date Field Formatting

Date fields come as ISO strings. Parse and format them for display.

```tsx
import {
  ComponentRendering,
  DateField,
} from '@sitecore-content-sdk/nextjs';

interface EventFields {
  EventDate: DateField;
}

interface EventProps {
  rendering: ComponentRendering;
  fields: EventFields;
}

const EventDate = (props: EventProps): JSX.Element => {
  const dateValue = props.fields.EventDate?.value;
  const formatted = dateValue
    ? new Date(dateValue).toLocaleDateString('en-US', {
        year: 'numeric',
        month: 'long',
        day: 'numeric',
      })
    : '';

  return (
    <time className="event-date" dateTime={dateValue || ''}>
      {formatted}
    </time>
  );
};

export default EventDate;
```

## SXA Patterns

### Component Variants

SXA component variants allow multiple visual presentations of the same component. In headless mode, the variant is passed as a rendering parameter and the front-end component decides how to render.

```tsx
import {
  Text,
  TextField,
  Image,
  ImageField,
  ComponentRendering,
} from '@sitecore-content-sdk/nextjs';

interface CardFields {
  Title: TextField;
  Image: ImageField;
}

interface CardProps {
  rendering: ComponentRendering;
  fields: CardFields;
}

const Card = (props: CardProps): JSX.Element => {
  const variant = props.rendering.params?.FieldNames || 'Default';

  switch (variant) {
    case 'Compact':
      return (
        <div className="card card--compact">
          <Text field={props.fields.Title} tag="h4" />
        </div>
      );

    case 'Featured':
      return (
        <div className="card card--featured">
          <Image field={props.fields.Image} className="card__image--large" />
          <Text field={props.fields.Title} tag="h2" />
        </div>
      );

    default:
      return (
        <div className="card card--default">
          <Image field={props.fields.Image} />
          <Text field={props.fields.Title} tag="h3" />
        </div>
      );
  }
};

export default Card;
```

### SXA Grid System Usage

SXA provides a 12-column grid. In headless mode, grid classes are applied via rendering parameters. The `ColumnWidthStyles` or `GridParameters` param carries the CSS class.

```tsx
import {
  Placeholder,
  ComponentRendering,
} from '@sitecore-content-sdk/nextjs';

interface ColumnSplitterProps {
  rendering: ComponentRendering;
}

const ColumnSplitter = (props: ColumnSplitterProps): JSX.Element => {
  const enabledPlaceholders = props.rendering.params?.EnabledPlaceholders || '';
  const placeholderNames = enabledPlaceholders.split(',').filter(Boolean);
  const columnWidths = (props.rendering.params?.ColumnWidths || '')
    .split(',')
    .filter(Boolean);

  return (
    <div className="row">
      {placeholderNames.map((phName: string, index: number) => (
        <div
          key={phName}
          className={`col-${columnWidths[index] || '12'}`}
        >
          <Placeholder name={phName.trim()} rendering={props.rendering} />
        </div>
      ))}
    </div>
  );
};

export default ColumnSplitter;
```

### Composite Components

Composite components combine a parent with multiple child placeholders. Common in SXA for layout building blocks like Tabs, Accordions, and Carousels.

```tsx
import {
  Text,
  TextField,
  Placeholder,
  ComponentRendering,
} from '@sitecore-content-sdk/nextjs';
import { useState } from 'react';

interface AccordionFields {
  Title: TextField;
}

interface AccordionProps {
  rendering: ComponentRendering;
  fields: AccordionFields;
}

const Accordion = (props: AccordionProps): JSX.Element => {
  const [openIndex, setOpenIndex] = useState<number | null>(0);
  const sectionCount = parseInt(props.rendering.params?.SectionCount || '3', 10);

  return (
    <div className="accordion">
      <Text field={props.fields.Title} tag="h2" />
      {Array.from({ length: sectionCount }, (_, i) => (
        <div key={i} className="accordion__section">
          <button
            className="accordion__trigger"
            onClick={() => setOpenIndex(openIndex === i ? null : i)}
            aria-expanded={openIndex === i}
          >
            Section {i + 1}
          </button>
          {openIndex === i && (
            <div className="accordion__panel">
              <Placeholder
                name={`accordion-section-${i}`}
                rendering={props.rendering}
              />
            </div>
          )}
        </div>
      ))}
    </div>
  );
};

export default Accordion;
```

## Traditional MVC Patterns (.NET)

### Controller Rendering

A controller rendering maps to a C# controller action. The controller retrieves the datasource item and returns a view.

```csharp
using Sitecore.Mvc.Controllers;
using Sitecore.Mvc.Presentation;
using Sitecore.Data.Items;
using System.Web.Mvc;

namespace Feature.Hero.Controllers
{
    public class HeroBannerController : SitecoreController
    {
        public ActionResult Index()
        {
            var rendering = RenderingContext.Current.Rendering;
            var datasource = rendering.Item;

            if (datasource == null)
            {
                return new EmptyResult();
            }

            var model = new HeroBannerViewModel
            {
                Heading = datasource["Heading"],
                Subheading = datasource["Subheading"],
                BackgroundImage = Sitecore.Resources.Media.MediaManager
                    .GetMediaUrl(((Sitecore.Data.Fields.ImageField)datasource.Fields["Background Image"]).MediaItem),
            };

            return View("~/Views/Feature/Hero/HeroBanner.cshtml", model);
        }
    }

    public class HeroBannerViewModel
    {
        public string Heading { get; set; }
        public string Subheading { get; set; }
        public string BackgroundImage { get; set; }
    }
}
```

### View Rendering

A view rendering is a `.cshtml` file that accesses the rendering datasource directly. Simpler than controller renderings but less testable.

```cshtml
@using Sitecore.Mvc
@using Sitecore.Mvc.Presentation
@using Sitecore.Data.Fields

@{
    var datasource = RenderingContext.Current.Rendering.Item;
    if (datasource == null) { return; }
}

<div class="promo-banner">
    <h2>@Html.Sitecore().Field("Heading", datasource)</h2>
    <div class="promo-banner__body">
        @Html.Sitecore().Field("Body", datasource)
    </div>
    @{
        LinkField link = datasource.Fields["Call To Action"];
        if (link != null && !string.IsNullOrEmpty(link.Url))
        {
            <a href="@link.Url" class="promo-banner__cta">
                @(string.IsNullOrEmpty(link.Text) ? "Learn More" : link.Text)
            </a>
        }
    }
</div>
```

### Glass Mapper Model Pattern

Glass Mapper provides strongly-typed models mapped to Sitecore templates. Define an interface matching the template, then inject via controller.

```csharp
using Glass.Mapper.Sc.Configuration.Attributes;
using System;

namespace Feature.Articles.Models
{
    [SitecoreType(TemplateId = "{A1B2C3D4-E5F6-7890-ABCD-EF1234567890}")]
    public class ArticleModel
    {
        [SitecoreField("Article Title")]
        public virtual string ArticleTitle { get; set; }

        [SitecoreField("Article Body")]
        public virtual string ArticleBody { get; set; }

        [SitecoreField("Publish Date")]
        public virtual DateTime PublishDate { get; set; }

        [SitecoreField("Thumbnail")]
        public virtual Glass.Mapper.Sc.Fields.Image Thumbnail { get; set; }

        [SitecoreField("Author")]
        public virtual ArticleAuthorModel Author { get; set; }
    }

    [SitecoreType(TemplateId = "{B2C3D4E5-F6A7-8901-BCDE-F12345678901}")]
    public class ArticleAuthorModel
    {
        [SitecoreField("Full Name")]
        public virtual string FullName { get; set; }

        [SitecoreField("Bio")]
        public virtual string Bio { get; set; }
    }
}
```

Controller using Glass Mapper:

```csharp
using Glass.Mapper.Sc.Web.Mvc;
using Feature.Articles.Models;
using System.Web.Mvc;

namespace Feature.Articles.Controllers
{
    public class ArticleController : GlassController
    {
        public ActionResult ArticleDetail()
        {
            var model = GetDataSourceItem<ArticleModel>();

            if (model == null)
            {
                return new EmptyResult();
            }

            return View("~/Views/Feature/Articles/ArticleDetail.cshtml", model);
        }
    }
}
```

### Helix Solution Structure

Organize the solution according to Helix principles:

```
src/
  Foundation/
    Foundation.DependencyInjection/    # IoC container setup
    Foundation.Serialization/          # Shared serialization config
    Foundation.ORM/                    # Glass Mapper or other ORM setup
  Feature/
    Feature.Navigation/                # Navigation components
    Feature.Hero/                      # Hero banner components
    Feature.Articles/                  # Article listing and detail
    Feature.Search/                    # Search components
  Project/
    Project.Website/                   # The Sitecore web project, wires everything together
    Project.Rendering/                 # Next.js rendering host (headless)
```

Rules:
- **Foundation** depends only on Sitecore APIs and external libraries.
- **Feature** depends on Foundation but never on other Features.
- **Project** depends on both Foundation and Feature; it wires them together.

## GraphQL Patterns

### Experience Edge Query for Content List

Query Experience Edge to fetch a list of content items. Commonly used in Next.js `getStaticProps` or server components.

```typescript
import { GraphQLClient, gql } from 'graphql-request';

const EDGE_ENDPOINT = process.env.SITECORE_EDGE_URL || '';
const EDGE_API_KEY = process.env.SITECORE_EDGE_API_KEY || '';

const GET_ARTICLES = gql`
  query GetArticles($language: String!, $first: Int) {
    search(
      where: {
        AND: [
          { name: "_templates", value: "{ARTICLE-TEMPLATE-ID}", operator: CONTAINS }
          { name: "_language", value: $language, operator: EQ }
        ]
      }
      first: $first
      orderBy: { name: "PublishDate", direction: DESC }
    ) {
      total
      results {
        id
        name
        url {
          path
        }
        ... on ArticlePage {
          articleTitle {
            value
          }
          articleSummary {
            value
          }
          publishDate {
            value
          }
          thumbnail {
            src
            alt
          }
        }
      }
    }
  }
`;

interface ArticleResult {
  id: string;
  name: string;
  url: { path: string };
  articleTitle: { value: string };
  articleSummary: { value: string };
  publishDate: { value: string };
  thumbnail: { src: string; alt: string };
}

export async function getArticles(
  language: string = 'en',
  first: number = 10
): Promise<ArticleResult[]> {
  const client = new GraphQLClient(EDGE_ENDPOINT, {
    headers: { sc_apikey: EDGE_API_KEY },
  });

  const data = await client.request<{ search: { results: ArticleResult[] } }>(
    GET_ARTICLES,
    { language, first }
  );

  return data.search.results;
}
```

### Integrated GraphQL in a Component

For components that need custom data beyond the layout data, use integrated GraphQL. The query is defined alongside the component and executed during rendering.

```graphql
# ArticleListing.graphql
query ArticleListingQuery($datasource: String!, $language: String!) {
  datasource: item(path: $datasource, language: $language) {
    ... on ArticleListingFolder {
      children(
        includeTemplateIDs: "{ARTICLE-TEMPLATE-ID}"
        first: 10
      ) {
        results {
          ... on ArticlePage {
            id
            url {
              path
            }
            articleTitle {
              value
            }
            publishDate {
              value
            }
          }
        }
      }
    }
  }
}
```

Component consuming the integrated query result:

```tsx
import Link from 'next/link';
import { ComponentRendering } from '@sitecore-content-sdk/nextjs';

interface ArticleItem {
  id: string;
  url: { path: string };
  articleTitle: { value: string };
  publishDate: { value: string };
}

interface ArticleListingData {
  datasource: {
    children: {
      results: ArticleItem[];
    };
  };
}

interface ArticleListingProps {
  rendering: ComponentRendering;
  fields: Record<string, unknown>;
  graphQLResult: ArticleListingData;
}

const ArticleListing = (props: ArticleListingProps): JSX.Element => {
  const articles = props.graphQLResult?.datasource?.children?.results || [];

  return (
    <div className="article-listing">
      <ul>
        {articles.map((article) => (
          <li key={article.id}>
            <Link href={article.url.path}>
              {article.articleTitle.value}
            </Link>
            <time>{new Date(article.publishDate.value).toLocaleDateString()}</time>
          </li>
        ))}
      </ul>
    </div>
  );
};

export default ArticleListing;
```

### GraphQL Query for Single Item by Path

Fetch a single item with all its fields by path. Useful for settings items or shared content.

```typescript
const GET_ITEM_BY_PATH = gql`
  query GetItemByPath($path: String!, $language: String!) {
    item(path: $path, language: $language) {
      id
      name
      template {
        name
      }
      fields {
        name
        value
        ... on ImageFieldValue {
          src
          alt
          width
          height
        }
        ... on LinkFieldValue {
          href
          text
          target
        }
      }
    }
  }
`;
```

## Naming Conventions

| Artifact | Convention | Example |
|---|---|---|
| Templates | PascalCase with spaces | `Article Page`, `Hero Banner` |
| Base Templates | Underscore prefix, PascalCase | `_HasTitle`, `_HasMetadata` |
| Template Fields | PascalCase | `Page Title`, `Hero Image` |
| Template Sections | PascalCase | `Content`, `Navigation`, `Metadata` |
| Renderings | PascalCase matching component name | `HeroBanner`, `ArticleCard` |
| Placeholder Keys | kebab-case | `main-content`, `sidebar-right` |
| Component files (Next.js) | PascalCase `.tsx` | `HeroBanner.tsx`, `ArticleCard.tsx` |
| Serialization modules | kebab-case | `content-articles`, `templates-foundation` |
| Content paths | PascalCase with spaces | `/sitecore/content/Site/Home/About Us` |
| Media Library folders | PascalCase with spaces | `/sitecore/media library/Project/Site Name` |
| Controller names (.NET) | PascalCase + `Controller` suffix | `HeroBannerController` |
| View files (.NET) | PascalCase `.cshtml` | `HeroBanner.cshtml` |
| Helix layers | Foundation, Feature, Project | `Feature.Navigation`, `Foundation.ORM` |
