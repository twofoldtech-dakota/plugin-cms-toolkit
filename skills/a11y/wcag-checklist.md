# WCAG 2.2 AA Checklist for CMS Projects

Complete checklist of WCAG 2.2 Level A and AA success criteria mapped to common CMS patterns.

Use this as a reference when scanning. Not every criterion applies to every file — match criteria to the file type being scanned.

---

## 1. Perceivable

### 1.1 Text Alternatives

| Criterion | Level | What to check in CMS code |
|-----------|-------|---------------------------|
| **1.1.1** Non-text Content | A | Every `<img>`, CMS image field helper, and media picker output has meaningful alt text. Decorative images use `alt=""` with `role="presentation"`. |

### 1.2 Time-based Media

| Criterion | Level | What to check |
|-----------|-------|---------------|
| **1.2.1** Audio-only / Video-only (Prerecorded) | A | Video/audio components have transcript or text alternative |
| **1.2.2** Captions (Prerecorded) | A | Video players include caption track support |
| **1.2.3** Audio Description or Media Alternative | A | Video has audio description track or text equivalent |
| **1.2.4** Captions (Live) | AA | Live media components support real-time captions |
| **1.2.5** Audio Description (Prerecorded) | AA | Video components support audio description toggle |

### 1.3 Adaptable

| Criterion | Level | What to check |
|-----------|-------|---------------|
| **1.3.1** Info and Relationships | A | Heading hierarchy is logical (`h1` → `h2` → `h3`). Lists use `<ul>`/`<ol>`. Tables use `<th>`. Forms use `<label>`. |
| **1.3.2** Meaningful Sequence | A | DOM order matches visual order. CSS doesn't reorder content in confusing ways. |
| **1.3.3** Sensory Characteristics | A | Instructions don't rely on shape, size, position, or sound alone ("click the round button") |
| **1.3.4** Orientation | AA | No CSS that locks to portrait or landscape (`orientation` media query with `width: 0`) |
| **1.3.5** Identify Input Purpose | AA | Form inputs use `autocomplete` attributes for common fields (name, email, address) |

### 1.4 Distinguishable

| Criterion | Level | What to check |
|-----------|-------|---------------|
| **1.4.1** Use of Color | A | Information not conveyed by color alone. Error states use icons/text, not just red. |
| **1.4.2** Audio Control | A | Auto-playing audio has pause/stop/mute control |
| **1.4.3** Contrast (Minimum) | AA | Text has 4.5:1 contrast ratio (3:1 for large text). Check CSS color/background combinations. |
| **1.4.4** Resize Text | AA | Text can resize to 200% without loss. No `max-height` with `overflow: hidden` on text containers. |
| **1.4.5** Images of Text | AA | Actual text used instead of text in images (except logos) |
| **1.4.10** Reflow | AA | Content reflows at 320px width without horizontal scroll. No `overflow-x: scroll` on text content. |
| **1.4.11** Non-text Contrast | AA | UI components and graphics have 3:1 contrast ratio against adjacent colors |
| **1.4.12** Text Spacing | AA | Content works with increased line-height (1.5x), letter-spacing (0.12em), word-spacing (0.16em) |
| **1.4.13** Content on Hover or Focus | AA | Tooltips/popovers are dismissible, hoverable, and persistent |

---

## 2. Operable

### 2.1 Keyboard Accessible

| Criterion | Level | What to check |
|-----------|-------|---------------|
| **2.1.1** Keyboard | A | All interactive elements reachable and operable via keyboard. Custom components have `onKeyDown`. |
| **2.1.2** No Keyboard Trap | A | Focus can always leave a component. Modals have close-on-Escape. |
| **2.1.4** Character Key Shortcuts | A | Single-character shortcuts can be turned off or remapped |

### 2.2 Enough Time

| Criterion | Level | What to check |
|-----------|-------|---------------|
| **2.2.1** Timing Adjustable | A | Session timeouts can be extended. Auto-logout has warning. |
| **2.2.2** Pause, Stop, Hide | A | Auto-updating content (carousels, tickers) has pause control |

### 2.3 Seizures and Physical Reactions

| Criterion | Level | What to check |
|-----------|-------|---------------|
| **2.3.1** Three Flashes or Below Threshold | A | No content flashes more than 3 times per second |

### 2.4 Navigable

| Criterion | Level | What to check |
|-----------|-------|---------------|
| **2.4.1** Bypass Blocks | A | Skip navigation link present. Landmark roles used (`<main>`, `<nav>`, `<aside>`). |
| **2.4.2** Page Titled | A | `<title>` is descriptive and unique per page. CMS page title field rendered in `<title>`. |
| **2.4.3** Focus Order | A | Tab order follows logical reading order |
| **2.4.4** Link Purpose (In Context) | A | Link text is descriptive. No "click here" or "read more" without context. |
| **2.4.5** Multiple Ways | AA | At least two ways to reach each page (nav + sitemap, search, etc.) |
| **2.4.6** Headings and Labels | AA | Headings describe topic. Labels describe purpose. |
| **2.4.7** Focus Visible | AA | Focus indicator is visible on all interactive elements. No `outline: none` without replacement. |
| **2.4.11** Focus Not Obscured (Minimum) | AA | Focused element not fully hidden by sticky headers, modals, or overlays |

### 2.5 Input Modalities

| Criterion | Level | What to check |
|-----------|-------|---------------|
| **2.5.1** Pointer Gestures | A | Multi-point gestures have single-pointer alternative |
| **2.5.2** Pointer Cancellation | A | Click actions fire on `mouseup`/`pointerup`, not `mousedown` |
| **2.5.3** Label in Name | A | Visible text label is included in the accessible name |
| **2.5.4** Motion Actuation | A | Shake/tilt actions have button alternative |
| **2.5.7** Dragging Movements | AA | Drag-and-drop has click alternative |
| **2.5.8** Target Size (Minimum) | AA | Interactive targets are at least 24x24 CSS pixels |

---

## 3. Understandable

### 3.1 Readable

| Criterion | Level | What to check |
|-----------|-------|---------------|
| **3.1.1** Language of Page | A | `<html lang="xx">` is set. CMS language detection feeds this attribute. |
| **3.1.2** Language of Parts | AA | Content in a different language is marked with `lang` attribute |

### 3.2 Predictable

| Criterion | Level | What to check |
|-----------|-------|---------------|
| **3.2.1** On Focus | A | Focus doesn't trigger unexpected context changes |
| **3.2.2** On Input | A | Changing form value doesn't auto-submit or navigate |
| **3.2.3** Consistent Navigation | AA | Navigation order is consistent across pages (check layout templates) |
| **3.2.4** Consistent Identification | AA | Same functionality has same label across pages |

### 3.3 Input Assistance

| Criterion | Level | What to check |
|-----------|-------|---------------|
| **3.3.1** Error Identification | A | Error messages identify the field and describe the error |
| **3.3.2** Labels or Instructions | A | Form fields have visible labels. Required fields are marked. |
| **3.3.3** Error Suggestion | AA | Error messages suggest how to fix the issue |
| **3.3.4** Error Prevention (Legal, Financial, Data) | AA | Submissions are reversible, verifiable, or confirmable |
| **3.3.7** Redundant Entry | A | Previously entered info is auto-populated or selectable |
| **3.3.8** Accessible Authentication (Minimum) | AA | Login doesn't require cognitive function test beyond password entry |

---

## 4. Robust

### 4.1 Compatible

| Criterion | Level | What to check |
|-----------|-------|---------------|
| **4.1.2** Name, Role, Value | A | Custom components expose name, role, and state via ARIA. `role`, `aria-label`, `aria-expanded`, etc. |
| **4.1.3** Status Messages | AA | Dynamic status changes use `aria-live` regions or `role="status"` / `role="alert"` |

---

## CMS Content Model Checklist

These are not WCAG criteria but critical for CMS projects to enable accessible content authoring:

| Check | Why |
|-------|-----|
| Every image field has a companion alt text field | Authors need a place to enter alt text |
| Alt text field is marked required when image is required | Prevents images without descriptions |
| Video/media fields have caption and transcript companion fields | Enables 1.2.x compliance |
| Link fields have a text/label companion field | Enables descriptive link text (2.4.4) |
| Rich text editor is configured with heading levels | Prevents heading hierarchy violations (1.3.1) |
| Content type has a page title field mapped to `<title>` | Enables unique page titles (2.4.2) |
| Language field or language branch support is configured | Enables language identification (3.1.1, 3.1.2) |
