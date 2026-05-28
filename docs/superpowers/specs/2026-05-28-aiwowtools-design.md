# AIWowTools — Design Spec
_Date: 2026-05-28_

## Overview

A free, no-signup online toolbox called **AIWowTools** with 16 fully working tools across two categories: Text Tools and Image Tools. Dark & premium design (same DNA as the Lumina AI landing page in this repo). Static HTML/CSS/JS — no build step, no frameworks, no server required.

---

## Tech Stack

| Concern | Solution |
|---------|----------|
| Delivery | One `.html` file per page — open directly in browser or serve statically |
| Animations | GSAP 3.12.5 + ScrollTrigger (CDN) |
| Smooth scroll | Lenis 1.1.14 (CDN) |
| Fonts | Space Grotesk (headings) + Inter (body) via Google Fonts |
| Image processing | HTML5 Canvas API (vanilla JS) |
| No frameworks, no build tools, no npm |

---

## Color Palette

| Role | Value |
|------|-------|
| Background | `#020617` |
| Surface | `#0f172a` |
| Border | `#1e293b` |
| Primary (Text tools) | `#6366f1` (indigo) |
| Primary Light | `#818cf8` |
| Secondary (Image tools) | `#a855f7` (purple) |
| Secondary Light | `#c084fc` |
| Gradient | `linear-gradient(135deg, #6366f1, #a855f7)` |
| Text Primary | `#f1f5f9` |
| Text Muted | `#64748b` |

---

## Typography

- **Headings:** Space Grotesk, weight 700–800
- **Body / UI labels:** Inter, weight 400–600
- **Hero headline:** `clamp(2.5rem, 6vw, 4.5rem)`, weight 800
- **Tool page title:** `clamp(1.5rem, 3vw, 2.25rem)`, weight 800
- **Card title:** `1rem`, weight 700

---

## File Structure

```
index.html                     ← Homepage
tools/
  word-counter.html
  case-converter.html
  lorem-ipsum.html
  text-diff.html
  remove-duplicates.html
  text-reverser.html
  whitespace-cleaner.html
  text-to-slug.html
  line-sorter.html
  find-replace.html
  image-resizer.html
  image-compressor.html
  format-converter.html
  image-to-base64.html
  grayscale-converter.html
  image-cropper.html
```

18 total files. Each file is self-contained with embedded CSS and JS.

---

## Homepage (`index.html`)

### 1. Navigation (Sticky)
- Logo: `⚡ AIWow` + `Tools` (indigo)
- Links: Text Tools · Image Tools · All Tools
- Right: search icon (opens search bar)
- Behavior: starts transparent, gains glassmorphism (`backdrop-filter: blur(12px)`) + border on scroll
- Fade-in on page load (GSAP, 0.6s)

### 2. Hero Section
- Badge: `⚡ 16 Free Tools · No Signup · 100% Private`
- Headline: "Your AI-Powered" (white) + "Free Toolbox" (indigo→purple gradient)
- Subtext: "Convert, transform & create. Instant results. No account needed."
- Search bar: text input + Search button — filters the tool grid live as user types
- Animations: badge fades in first, headline words stagger in, subtext + search fade in after

### 3. Category Tabs
- Three pills: **All (16)** · **🔤 Text (10)** · **🖼️ Image (6)**
- Active pill: indigo→purple gradient background
- Inactive: dark surface + border
- Click switches the visible tool cards with GSAP fade (opacity + scale)

### 4. Tool Grid
- Responsive: 4 cols (desktop) → 3 cols (tablet) → 2 cols (mobile) → 1 col (small mobile)
- Each card contains:
  - Emoji icon (top)
  - Category badge (indigo for Text, purple for Image)
  - Tool name (bold)
  - One-line description (muted)
  - Full card is a link → `tools/<slug>.html`
- Scroll-triggered stagger reveal: cards fade in + scale from `0.9 → 1` with `0.08s` stagger
- Hover: card lifts `6px`, border switches to indigo/purple glow, shimmer sweep left→right
- `pointer-events: none` on `::before` / `::after` pseudo-elements so link clicks pass through

### 5. Stats Bar
- Four stats: `16+` Free Tools · `100%` Private · `0` Signups · `Free` Forever
- Numbers count up from 0 when scrolled into view (GSAP counter)
- Glassmorphism card, indigo separator lines

### 6. Footer
- Left: logo + tagline "The only toolbox you'll ever need."
- Center: Quick Links (Text Tools, Image Tools, All Tools)
- Right: About · Privacy · Contact (placeholder `#` links)
- Top border: 1px gradient line (indigo → purple)
- Copyright: `© 2026 AIWowTools. Free forever.`

---

## Individual Tool Pages

All 16 tool pages share the same shell structure. Only the tool interface block differs.

### Shared Shell

**Navigation** — identical to homepage nav, with `../index.html` as logo href

**Breadcrumb**
```
⚡ AIWowTools  ›  Text Tools  ›  Word Counter
```
Links: logo → `../index.html`, category → `../index.html#text` (filtered)

**Tool Header**
- Large emoji icon (left)
- Tool name (h1, gradient text)
- One-line description
- Category badge (indigo for Text / purple for Image)

**Tool Interface** — see patterns below

**Related Tools** — 3 cards from the same category at the bottom, scroll-triggered reveal

**Footer** — same as homepage footer (relative paths back to `../index.html`)

---

## Tool Interface Patterns

### Pattern A — Text Tool
Used by all 10 text tools.

```
┌─────────────────────────────────────────┐
│  Input Label                            │
│  ┌───────────────────────────────────┐  │
│  │ textarea (min 160px, resizable)   │  │
│  └───────────────────────────────────┘  │
│  [Primary Action]  [Clear]  [Copy]      │
└─────────────────────────────────────────┘
┌─────────────────────────────────────────┐
│  Output / Results                       │
│  (stat cards grid OR output textarea)   │
└─────────────────────────────────────────┘
```

- Primary action button: gradient indigo→purple
- Results update in real-time where appropriate (Word Counter, Case Converter, Slug, etc.)
- Copy button copies result to clipboard, shows `✓ Copied!` for 2s

### Pattern B — Image Tool
Used by all 6 image tools.

```
┌─────────────────────────────────────────┐
│  Drag & Drop Zone                       │
│  📁 Drop image or click to upload       │
│  PNG, JPG, WebP up to 10MB              │
└─────────────────────────────────────────┘
┌─────────────────────────────────────────┐
│  Settings Panel (tool-specific)         │
│  (inputs, sliders, selects)             │
└─────────────────────────────────────────┘
┌─────────────────────────────────────────┐
│  Preview (canvas element)               │
└─────────────────────────────────────────┘
[Process & Download]
```

- Canvas API handles all image processing client-side
- Preview updates on setting change after image is loaded
- Download triggers `<a download>` with canvas `toBlob()`

---

## The 16 Tools

### Text Tools

| # | File | Tool Name | Core Behavior |
|---|------|-----------|---------------|
| 1 | `word-counter.html` | Word Counter | Real-time: words, characters (with/without spaces), sentences, paragraphs, read time |
| 2 | `case-converter.html` | Case Converter | 6 modes: UPPERCASE, lowercase, Title Case, camelCase, snake_case, kebab-case. One-click copy each. |
| 3 | `lorem-ipsum.html` | Lorem Ipsum Generator | Number input (1–20) + unit selector (paragraphs/sentences/words) → generate button → copyable output |
| 4 | `text-diff.html` | Text Diff | Two side-by-side textareas → Compare button → line-by-line diff with green additions / red removals |
| 5 | `remove-duplicates.html` | Remove Duplicates | Textarea input → Remove button → deduplicated output. Options: case-sensitive, trim whitespace |
| 6 | `text-reverser.html` | Text Reverser | Two mode buttons: "Reverse All Characters" and "Reverse Word Order". Real-time output. |
| 7 | `whitespace-cleaner.html` | Whitespace Cleaner | Checkboxes: trim leading/trailing, normalize multiple spaces to one, remove blank lines. Real-time. |
| 8 | `text-to-slug.html` | Text to Slug | Real-time: converts input to lowercase-hyphenated-slug, strips special chars, copy button |
| 9 | `line-sorter.html` | Line Sorter | Four sort buttons: A→Z, Z→A, by length (asc/desc), shuffle. Input/output textareas. |
| 10 | `find-replace.html` | Find & Replace | Three fields: input text, find term, replace with. Replace All button. Match count display. |

### Image Tools

| # | File | Tool Name | Core Behavior |
|---|------|-----------|---------------|
| 11 | `image-resizer.html` | Image Resizer | Width + height number inputs, lock-aspect-ratio toggle, canvas resize, download PNG |
| 12 | `image-compressor.html` | Image Compressor | Quality slider (1–100), before/after file size display, download compressed JPG |
| 13 | `format-converter.html` | Format Converter | Target format selector (PNG/JPG/WebP), canvas `toBlob()` conversion, download |
| 14 | `image-to-base64.html` | Image to Base64 | Upload → Base64 string in textarea → copy button. Reverse: paste Base64 → preview image |
| 15 | `grayscale-converter.html` | Grayscale Converter | Canvas `getImageData` → desaturate each pixel → preview → download |
| 16 | `image-cropper.html` | Image Cropper | Mouse drag on canvas to select crop rectangle → crop button → download cropped image |

---

## Animations & Interactions

### Global
- **Lenis** smooth scroll wraps the entire page
- **GSAP ScrollTrigger** syncs to Lenis virtual scroll
- **`prefers-reduced-motion`** disables parallax and complex animations, keeps fade-ins only

### Homepage
- Nav fades in on load (0.6s)
- Hero: badge → headline (word stagger) → subtext → search — sequential GSAP timeline
- Tool cards: scroll-triggered stagger reveal, `scale: 0.9 → 1`, `opacity: 0 → 1`, 0.08s stagger
- Category tab switch: GSAP `opacity: 0 → 1` + `y: 8 → 0` on newly visible cards
- Stats: counter animation when scrolled into view

### Tool Pages
- Tool header fades in on load
- Interface panels slide up with `y: 20 → 0` on load
- Related tools: scroll-triggered stagger reveal (same as homepage cards)

### Cards
- Hover: `translateY(-6px)`, border-color → indigo/purple, `box-shadow: 0 0 30px rgba(99,102,241,0.2)`
- `::before` and `::after` pseudo-elements always have `pointer-events: none`

---

## Responsiveness

- **Desktop** (>1024px): 4-col tool grid, full nav
- **Tablet** (768–1024px): 3-col grid
- **Mobile** (480–768px): 2-col grid, hamburger nav
- **Small mobile** (<480px): 1-col grid
- Image tools on mobile: vertical stack (drop zone → settings → preview)
- Custom cursor disabled on touch devices

---

## Constraints

- All processing is **client-side only** — no server, no API calls
- Image tools are limited to file sizes the browser Canvas API can handle comfortably (~10MB)
- No actual AI features — "AIWow" is a brand name, not a technical description
- No user accounts, no data storage, no cookies
