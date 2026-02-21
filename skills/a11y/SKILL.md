---
name: a11y
description: >
  WCAG 2.2 AA accessibility scanner for Sitecore, Umbraco, and Optimizely CMS projects.
  Scans Razor views, React/Next.js components, JSX templates, and content type definitions
  for accessibility violations. Use when asked to: scan for accessibility issues, run an
  accessibility audit, check WCAG compliance, find a11y problems, or audit templates for
  accessibility. Triggers on "accessibility", "a11y", "WCAG", "screen reader", "aria".
user-invocable: true
argument-hint: "[scope: all | views | components | content-types]"
allowed-tools: Read, Grep, Glob, Bash
---

# CMS Accessibility Scanner

Scan CMS projects for WCAG 2.2 AA violations across views, components, and content type definitions.

## Usage

```
/plugin-cms-toolkit:a11y                    # Full scan
/plugin-cms-toolkit:a11y views              # Razor views and templates only
/plugin-cms-toolkit:a11y components         # React/Next.js components only
/plugin-cms-toolkit:a11y content-types      # Content model accessibility only
```

## Workflow

### 1. Detect CMS and Gather Files

Determine the CMS platform (same detection as cms-detect), then collect files to scan:

**Sitecore:**
- `src/components/**/*.tsx` — Content SDK / JSS components
- `Views/**/*.cshtml` — MVC Razor views
- Template serialization YAML for content type field analysis

**Umbraco:**
- `Views/**/*.cshtml` — Razor views and partials
- `Views/Partials/blocklist/**/*.cshtml` — Block List component views
- Document type classes (`Models/**/*.cs`) for field analysis

**Optimizely:**
- `Views/**/*.cshtml` — Razor views
- `src/**/*.tsx` — JS SDK React components
- Content type classes (`Models/**/*.cs`) for field analysis

### 2. Scan for Violations

Read the full WCAG 2.2 AA checklist from [wcag-checklist.md](wcag-checklist.md) before scanning.

For each file, check against every applicable criterion. Group findings by WCAG principle.

#### Markup and Component Checks

**Perceivable (WCAG 1.x)**
- Images without alt text or empty alt on non-decorative images
- CMS image fields rendered without alt text binding (`<img>` without `alt={props.fields.Alt}`)
- Video/audio without captions or transcripts
- Information conveyed by color alone (CSS patterns like `.error { color: red }` without icon/text)
- Text embedded in images
- Missing language attributes on `<html>`
- Insufficient contrast patterns (e.g., light gray text classes, low-contrast CSS custom properties)

**Operable (WCAG 2.x)**
- Click handlers on non-interactive elements (`<div onClick>`, `<span onClick>`) without `role`, `tabIndex`, and keyboard handlers
- Missing skip navigation links
- Form elements without visible labels
- Custom components missing keyboard support (`onKeyDown`/`onKeyPress` alongside `onClick`)
- Focus traps without escape mechanism
- Motion/animation without `prefers-reduced-motion` media query
- Touch targets smaller than 24x24 CSS pixels

**Understandable (WCAG 3.x)**
- Forms without error identification and suggestion
- Missing `<label>` elements or `aria-label`/`aria-labelledby` on inputs
- Language not specified on page or content sections
- Inconsistent navigation patterns across templates

**Robust (WCAG 4.x)**
- Invalid ARIA roles or properties
- ARIA attributes on elements that don't support them
- Missing `role` attributes on custom interactive components
- Duplicate IDs in templates

#### Content Type Definition Checks

- Image fields without a corresponding alt text field (e.g., `HeroImage` exists but no `HeroImageAlt`)
- Rich text fields without guidance for editors on accessible content
- Link fields without text/label companion fields
- Video/media fields without caption or transcript companion fields
- Missing alt text in media picker configurations

#### CMS-Specific Checks

**Sitecore:**
- `<Image>` field helper used without `alt` field binding
- `<Link>` field helper without accessible text
- Placeholder components missing landmark roles
- Experience Editor markup compatibility (editing wrappers breaking semantics)

**Umbraco:**
- `MediaPicker` properties without alt text property on media type
- Block List/Grid renderings missing landmark structure
- `Html.PropertyFor()` calls on images without alt output
- Content Delivery API responses missing alt text fields

**Optimizely:**
- `ContentArea` rendering without landmark wrappers
- `@Html.PropertyFor()` on image properties without alt text
- Block views missing heading hierarchy
- Visual Builder components without ARIA announcements for dynamic content

### 3. Report

Output a structured report:

```
# Accessibility Scan Results

**Platform:** [detected CMS]
**Standard:** WCAG 2.2 AA
**Files scanned:** [count]
**Issues found:** [count by severity]

## Critical (WCAG A violations)

### [1.1.1] Non-text Content
**file:line** — `<img>` rendered without alt text
```tsx
// Current
<img src={props.fields.Image.value.src} />

// Fix
<img src={props.fields.Image.value.src} alt={props.fields.Image.value.alt} />
```

## Serious (WCAG AA violations)

### [1.4.3] Contrast (Minimum)
**file:line** — Description of issue
Fix: Specific remediation

## Moderate (Best practice)

...

## Content Model Issues

| Content Type | Issue | Recommendation |
|---|---|---|
| HeroBanner | Image field has no alt text companion | Add `HeroImageAlt` (Single-Line Text, required) |

## Summary

| Severity | Count |
|----------|-------|
| Critical (A) | X |
| Serious (AA) | X |
| Moderate (best practice) | X |
| Content model | X |

**Top 3 recommendations:**
1. ...
2. ...
3. ...
```

### Severity Levels

- **Critical** — WCAG 2.2 Level A failure. Blocks users with disabilities entirely.
- **Serious** — WCAG 2.2 Level AA failure. Significant barrier.
- **Moderate** — Best practice violation. Not a conformance failure but degrades experience.
