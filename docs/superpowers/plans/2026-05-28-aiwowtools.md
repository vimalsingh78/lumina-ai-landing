# AIWowTools Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build AIWowTools — a free, no-signup toolbox with 16 working tools (10 Text + 6 Image) in a dark premium design, delivered as static HTML/CSS/JS files.

**Architecture:** `index.html` homepage with search + category filter + tool grid, plus one self-contained HTML page per tool in `tools/`. Every page embeds its own CSS and JS — no shared files, no build step. Image tools use the HTML5 Canvas API for client-side processing.

**Tech Stack:** GSAP 3.12.5 + ScrollTrigger (CDN), Lenis 1.1.14 (CDN), Space Grotesk + Inter (Google Fonts), HTML5 Canvas API, vanilla JS.

---

## File Structure

```
index.html
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

---

## SHARED CSS (used in every task — copy verbatim into every `<style>` block)

```css
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
:root {
  --bg: #020617; --surface: #0f172a; --border: #1e293b;
  --primary: #6366f1; --primary-light: #818cf8;
  --secondary: #a855f7; --secondary-light: #c084fc;
  --gradient: linear-gradient(135deg, #6366f1, #a855f7);
  --text: #f1f5f9; --muted: #64748b;
}
body { background: var(--bg); color: var(--text); font-family: 'Inter', sans-serif; line-height: 1.6; min-height: 100vh; }
h1,h2,h3,h4 { font-family: 'Space Grotesk', sans-serif; }
a { color: inherit; text-decoration: none; }
/* NAV */
nav { position: sticky; top: 0; z-index: 100; padding: 0 24px; height: 60px; display: flex; align-items: center; gap: 24px; transition: background 0.3s, border-color 0.3s; }
nav.scrolled { background: rgba(15,23,42,0.85); backdrop-filter: blur(12px); border-bottom: 1px solid var(--border); }
.nav-logo { font-family: 'Space Grotesk', sans-serif; font-weight: 800; font-size: 1.1rem; }
.nav-logo span { color: var(--primary); }
.nav-links { display: flex; gap: 20px; margin-left: 24px; }
.nav-links a { font-size: 0.875rem; color: var(--muted); transition: color 0.2s; }
.nav-links a:hover { color: var(--text); }
.nav-cta { margin-left: auto; background: var(--gradient); color: #fff; font-size: 0.8rem; font-weight: 700; padding: 7px 16px; border-radius: 20px; }
/* FOOTER */
footer { border-top: 1px solid; border-image: var(--gradient) 1; padding: 40px 24px 24px; margin-top: 80px; }
.footer-inner { max-width: 1100px; margin: 0 auto; display: grid; grid-template-columns: 1fr 1fr 1fr; gap: 32px; }
.footer-logo { font-family: 'Space Grotesk', sans-serif; font-weight: 800; font-size: 1rem; margin-bottom: 6px; }
.footer-logo span { color: var(--primary); }
.footer-tagline { font-size: 0.8rem; color: var(--muted); }
.footer-col h4 { font-size: 0.75rem; text-transform: uppercase; letter-spacing: 0.08em; color: var(--muted); margin-bottom: 12px; }
.footer-col a { display: block; font-size: 0.85rem; color: var(--muted); margin-bottom: 6px; transition: color 0.2s; }
.footer-col a:hover { color: var(--text); }
.footer-copy { text-align: center; font-size: 0.75rem; color: var(--muted); margin-top: 32px; padding-top: 20px; border-top: 1px solid var(--border); }
/* BUTTONS */
.btn { cursor: pointer; border: none; font-family: 'Inter', sans-serif; font-weight: 600; border-radius: 8px; transition: opacity 0.2s, transform 0.15s; }
.btn:hover { opacity: 0.9; transform: translateY(-1px); }
.btn-primary { background: var(--gradient); color: #fff; padding: 10px 20px; font-size: 0.9rem; }
.btn-secondary { background: var(--surface); border: 1px solid var(--border); color: var(--text); padding: 10px 16px; font-size: 0.875rem; }
/* TOOL CARD (homepage + related) */
.tool-card { background: var(--surface); border: 1px solid var(--border); border-radius: 14px; padding: 20px; display: flex; flex-direction: column; gap: 8px; position: relative; overflow: hidden; transition: transform 0.25s, border-color 0.25s, box-shadow 0.25s; cursor: pointer; opacity: 0; }
.tool-card::before { content: ''; position: absolute; inset: 0; background: linear-gradient(135deg, rgba(99,102,241,0.06), transparent 60%); opacity: 0; transition: opacity 0.3s; pointer-events: none; }
.tool-card::after { content: ''; position: absolute; top: 0; left: -100%; width: 60%; height: 100%; background: linear-gradient(90deg, transparent, rgba(255,255,255,0.04), transparent); transition: left 0.5s; pointer-events: none; }
.tool-card:hover { transform: translateY(-6px); box-shadow: 0 0 30px rgba(99,102,241,0.2); border-color: var(--primary); }
.tool-card:hover::before { opacity: 1; }
.tool-card:hover::after { left: 150%; }
.tool-card[data-cat="image"]:hover { border-color: var(--secondary); box-shadow: 0 0 30px rgba(168,85,247,0.2); }
.tool-icon { font-size: 1.8rem; }
.tool-badge { display: inline-block; font-size: 0.7rem; font-weight: 700; padding: 2px 8px; border-radius: 10px; }
.tool-badge.text { background: rgba(99,102,241,0.15); color: var(--primary-light); }
.tool-badge.image { background: rgba(168,85,247,0.15); color: var(--secondary-light); }
.tool-name { font-family: 'Space Grotesk', sans-serif; font-weight: 700; font-size: 0.95rem; }
.tool-desc { font-size: 0.8rem; color: var(--muted); }
/* TEXTAREA / INPUT */
textarea, input[type="text"], input[type="number"] { background: var(--bg); border: 1px solid var(--border); color: var(--text); font-family: 'Inter', sans-serif; font-size: 0.9rem; border-radius: 8px; padding: 10px 12px; width: 100%; resize: vertical; transition: border-color 0.2s; outline: none; }
textarea:focus, input:focus { border-color: var(--primary); }
select { background: var(--surface); border: 1px solid var(--border); color: var(--text); font-family: 'Inter', sans-serif; font-size: 0.875rem; border-radius: 8px; padding: 8px 12px; cursor: pointer; outline: none; }
/* COPY FEEDBACK */
.copy-feedback { font-size: 0.75rem; color: #34d399; opacity: 0; transition: opacity 0.3s; margin-left: 8px; }
/* LENIS */
html.lenis { height: auto; }
.lenis.lenis-smooth { scroll-behavior: auto; }
/* REDUCED MOTION */
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after { animation-duration: 0.01ms !important; transition-duration: 0.01ms !important; }
}
```

## SHARED HEAD (CDNs — paste into every `<head>`)

```html
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@400;500;600;700;800&family=Inter:wght@400;500;600&display=swap" rel="stylesheet">
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.5/gsap.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.5/ScrollTrigger.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/lenis@1.1.14/dist/lenis.min.js"></script>
```

## SHARED LENIS + NAV SCROLL JS (paste into every page's `<script>`)

```js
// Lenis smooth scroll
const lenis = new Lenis();
lenis.on('scroll', ScrollTrigger.update);
gsap.ticker.add(time => lenis.raf(time * 1000));
gsap.ticker.lagSmoothing(0);
gsap.registerPlugin(ScrollTrigger);
// Nav glass on scroll
const nav = document.querySelector('nav');
window.addEventListener('scroll', () => nav.classList.toggle('scrolled', window.scrollY > 20));
```

---

## Task 1: Homepage — HTML Scaffold + CSS + Nav + Hero

**Files:**
- Create: `index.html`

- [ ] **Step 1: Create `index.html` with the full HTML skeleton**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <title>AIWowTools — Free Online Tools. No Signup.</title>
  <!-- SHARED HEAD (paste CDN block from plan header) -->
  <style>
    /* SHARED CSS (paste entire block from plan header) */

    /* HOMEPAGE-SPECIFIC */
    .hero { min-height: 85vh; display: flex; flex-direction: column; align-items: center; justify-content: center; text-align: center; padding: 60px 24px 40px; background: radial-gradient(ellipse at 50% 0%, rgba(99,102,241,0.18) 0%, transparent 65%); }
    #hero-badge { display: inline-flex; align-items: center; gap: 6px; background: rgba(99,102,241,0.12); border: 1px solid rgba(99,102,241,0.3); color: var(--primary-light); font-size: 0.8rem; font-weight: 600; padding: 6px 14px; border-radius: 20px; margin-bottom: 20px; }
    .hero-title { font-size: clamp(2.5rem, 6vw, 4.5rem); font-weight: 800; line-height: 1.1; margin-bottom: 16px; }
    .hero-title .gradient { background: var(--gradient); -webkit-background-clip: text; -webkit-text-fill-color: transparent; background-clip: text; }
    .hero-sub { font-size: 1.05rem; color: var(--muted); max-width: 500px; margin-bottom: 28px; }
    .search-bar { display: flex; gap: 8px; width: 100%; max-width: 520px; }
    .search-bar input { flex: 1; }
    .search-bar .btn { white-space: nowrap; }
    /* TABS */
    .tabs { display: flex; gap: 10px; padding: 0 24px; max-width: 1100px; margin: 0 auto 28px; flex-wrap: wrap; }
    .tab-btn { background: var(--surface); border: 1px solid var(--border); color: var(--muted); font-family: 'Space Grotesk', sans-serif; font-size: 0.85rem; font-weight: 600; padding: 8px 18px; border-radius: 20px; cursor: pointer; transition: all 0.2s; }
    .tab-btn.active { background: var(--gradient); border-color: transparent; color: #fff; }
    /* GRID */
    .tools-section { max-width: 1100px; margin: 0 auto; padding: 0 24px 60px; }
    .tools-grid { display: grid; grid-template-columns: repeat(4, 1fr); gap: 16px; }
    @media (max-width: 1024px) { .tools-grid { grid-template-columns: repeat(3, 1fr); } }
    @media (max-width: 768px) { .tools-grid { grid-template-columns: repeat(2, 1fr); } .hero-title { font-size: clamp(2rem, 8vw, 3rem); } }
    @media (max-width: 480px) { .tools-grid { grid-template-columns: 1fr; } }
    /* STATS */
    .stats { max-width: 1100px; margin: 0 auto 60px; padding: 0 24px; }
    .stats-inner { background: rgba(15,23,42,0.7); border: 1px solid var(--border); border-radius: 16px; padding: 28px; display: grid; grid-template-columns: repeat(4, 1fr); gap: 16px; backdrop-filter: blur(8px); }
    .stat-item { text-align: center; position: relative; }
    .stat-item:not(:last-child)::after { content: ''; position: absolute; right: 0; top: 20%; height: 60%; width: 1px; background: var(--border); }
    .stat-num { font-family: 'Space Grotesk', sans-serif; font-size: 2rem; font-weight: 800; background: var(--gradient); -webkit-background-clip: text; -webkit-text-fill-color: transparent; background-clip: text; }
    .stat-label { font-size: 0.8rem; color: var(--muted); margin-top: 4px; }
    @media (max-width: 600px) { .stats-inner { grid-template-columns: repeat(2, 1fr); } }
  </style>
</head>
<body>
  <nav>
    <div class="nav-logo">⚡ AIWow<span>Tools</span></div>
    <div class="nav-links">
      <a href="#" data-filter="text">Text Tools</a>
      <a href="#" data-filter="image">Image Tools</a>
      <a href="#" data-filter="all">All Tools</a>
    </div>
    <a href="#tools" class="nav-cta btn">Browse Tools</a>
  </nav>

  <!-- HERO -->
  <section class="hero">
    <div id="hero-badge">⚡ 16 Free Tools · No Signup · 100% Private</div>
    <h1 class="hero-title">Your AI-Powered<br><span class="gradient">Free Toolbox</span></h1>
    <p class="hero-sub">Convert, transform &amp; create. Instant results. No account needed.</p>
    <div class="search-bar">
      <input type="text" id="search-input" placeholder="🔍 Search tools... (e.g. word counter, image resize)">
      <button class="btn btn-primary" id="search-btn">Search</button>
    </div>
  </section>

  <!-- CATEGORY TABS -->
  <div id="tools" class="tabs">
    <button class="tab-btn active" data-cat="all">All (16)</button>
    <button class="tab-btn" data-cat="text">🔤 Text (10)</button>
    <button class="tab-btn" data-cat="image">🖼️ Image (6)</button>
  </div>

  <!-- TOOL GRID -->
  <section class="tools-section">
    <div class="tools-grid" id="tools-grid">
      <!-- Text Tools -->
      <a href="tools/word-counter.html" class="tool-card" data-cat="text">
        <div class="tool-icon">🔤</div>
        <span class="tool-badge text">Text</span>
        <div class="tool-name">Word Counter</div>
        <div class="tool-desc">Count words, chars, sentences &amp; read time</div>
      </a>
      <a href="tools/case-converter.html" class="tool-card" data-cat="text">
        <div class="tool-icon">🔡</div>
        <span class="tool-badge text">Text</span>
        <div class="tool-name">Case Converter</div>
        <div class="tool-desc">UPPER, lower, Title, camelCase, snake_case</div>
      </a>
      <a href="tools/lorem-ipsum.html" class="tool-card" data-cat="text">
        <div class="tool-icon">📝</div>
        <span class="tool-badge text">Text</span>
        <div class="tool-name">Lorem Ipsum</div>
        <div class="tool-desc">Generate placeholder text instantly</div>
      </a>
      <a href="tools/text-diff.html" class="tool-card" data-cat="text">
        <div class="tool-icon">🔀</div>
        <span class="tool-badge text">Text</span>
        <div class="tool-name">Text Diff</div>
        <div class="tool-desc">Compare two texts, highlight changes</div>
      </a>
      <a href="tools/remove-duplicates.html" class="tool-card" data-cat="text">
        <div class="tool-icon">🧹</div>
        <span class="tool-badge text">Text</span>
        <div class="tool-name">Remove Duplicates</div>
        <div class="tool-desc">Deduplicate lines from any text</div>
      </a>
      <a href="tools/text-reverser.html" class="tool-card" data-cat="text">
        <div class="tool-icon">🔄</div>
        <span class="tool-badge text">Text</span>
        <div class="tool-name">Text Reverser</div>
        <div class="tool-desc">Reverse characters or word order</div>
      </a>
      <a href="tools/whitespace-cleaner.html" class="tool-card" data-cat="text">
        <div class="tool-icon">✂️</div>
        <span class="tool-badge text">Text</span>
        <div class="tool-name">Whitespace Cleaner</div>
        <div class="tool-desc">Trim, normalize spaces &amp; blank lines</div>
      </a>
      <a href="tools/text-to-slug.html" class="tool-card" data-cat="text">
        <div class="tool-icon">🔗</div>
        <span class="tool-badge text">Text</span>
        <div class="tool-name">Text to Slug</div>
        <div class="tool-desc">Generate URL-friendly slugs instantly</div>
      </a>
      <a href="tools/line-sorter.html" class="tool-card" data-cat="text">
        <div class="tool-icon">📋</div>
        <span class="tool-badge text">Text</span>
        <div class="tool-name">Line Sorter</div>
        <div class="tool-desc">Sort A→Z, Z→A, by length, or shuffle</div>
      </a>
      <a href="tools/find-replace.html" class="tool-card" data-cat="text">
        <div class="tool-icon">🔍</div>
        <span class="tool-badge text">Text</span>
        <div class="tool-name">Find &amp; Replace</div>
        <div class="tool-desc">Bulk find and replace any text</div>
      </a>
      <!-- Image Tools -->
      <a href="tools/image-resizer.html" class="tool-card" data-cat="image">
        <div class="tool-icon">🖼️</div>
        <span class="tool-badge image">Image</span>
        <div class="tool-name">Image Resizer</div>
        <div class="tool-desc">Resize to any width &amp; height</div>
      </a>
      <a href="tools/image-compressor.html" class="tool-card" data-cat="image">
        <div class="tool-icon">🗜️</div>
        <span class="tool-badge image">Image</span>
        <div class="tool-name">Image Compressor</div>
        <div class="tool-desc">Reduce file size with quality control</div>
      </a>
      <a href="tools/format-converter.html" class="tool-card" data-cat="image">
        <div class="tool-icon">🎨</div>
        <span class="tool-badge image">Image</span>
        <div class="tool-name">Format Converter</div>
        <div class="tool-desc">Convert PNG ↔ JPG ↔ WebP</div>
      </a>
      <a href="tools/image-to-base64.html" class="tool-card" data-cat="image">
        <div class="tool-icon">💾</div>
        <span class="tool-badge image">Image</span>
        <div class="tool-name">Image to Base64</div>
        <div class="tool-desc">Encode images to Base64 strings</div>
      </a>
      <a href="tools/grayscale-converter.html" class="tool-card" data-cat="image">
        <div class="tool-icon">⚫</div>
        <span class="tool-badge image">Image</span>
        <div class="tool-name">Grayscale Converter</div>
        <div class="tool-desc">Convert images to black &amp; white</div>
      </a>
      <a href="tools/image-cropper.html" class="tool-card" data-cat="image">
        <div class="tool-icon">✂️</div>
        <span class="tool-badge image">Image</span>
        <div class="tool-name">Image Cropper</div>
        <div class="tool-desc">Drag to select and crop any image</div>
      </a>
    </div>
  </section>

  <!-- STATS BAR -->
  <section class="stats">
    <div class="stats-inner">
      <div class="stat-item"><div class="stat-num" data-target="16" data-suffix="+">0</div><div class="stat-label">Free Tools</div></div>
      <div class="stat-item"><div class="stat-num" data-target="100" data-suffix="%">0</div><div class="stat-label">Private</div></div>
      <div class="stat-item"><div class="stat-num" data-target="0" data-suffix="">0</div><div class="stat-label">Signups Required</div></div>
      <div class="stat-item"><div class="stat-num" data-target="0" data-suffix=" Cost">0</div><div class="stat-label">Always Free</div></div>
    </div>
  </section>

  <!-- FOOTER -->
  <footer>
    <div class="footer-inner">
      <div>
        <div class="footer-logo">⚡ AIWow<span>Tools</span></div>
        <p class="footer-tagline">The only toolbox you'll ever need.</p>
      </div>
      <div class="footer-col">
        <h4>Tools</h4>
        <a href="#" onclick="document.querySelector('[data-cat=text]').click();return false;">Text Tools</a>
        <a href="#" onclick="document.querySelector('[data-cat=image]').click();return false;">Image Tools</a>
        <a href="#tools">All Tools</a>
      </div>
      <div class="footer-col">
        <h4>Company</h4>
        <a href="#">About</a>
        <a href="#">Privacy</a>
        <a href="#">Contact</a>
      </div>
    </div>
    <p class="footer-copy">© 2026 AIWowTools. Free forever.</p>
  </footer>

  <script>
    // SHARED LENIS + NAV SCROLL JS (paste shared block from plan header)

    // GSAP load animations
    gsap.from('nav', { opacity: 0, y: -20, duration: 0.6, ease: 'power2.out' });
    const tl = gsap.timeline({ delay: 0.2 });
    tl.from('#hero-badge', { opacity: 0, y: 20, duration: 0.5 })
      .from('.hero-title', { opacity: 0, y: 30, duration: 0.6 }, '-=0.2')
      .from('.hero-sub',   { opacity: 0, y: 20, duration: 0.5 }, '-=0.3')
      .from('.search-bar', { opacity: 0, y: 20, duration: 0.5 }, '-=0.3');

    // Category tab filter
    const tabBtns = document.querySelectorAll('.tab-btn');
    const toolCards = document.querySelectorAll('.tool-card');
    tabBtns.forEach(btn => {
      btn.addEventListener('click', () => {
        tabBtns.forEach(b => b.classList.remove('active'));
        btn.classList.add('active');
        const cat = btn.dataset.cat;
        toolCards.forEach(card => {
          const show = cat === 'all' || card.dataset.cat === cat;
          if (show) {
            card.style.display = 'flex';
            gsap.to(card, { opacity: 1, scale: 1, y: 0, duration: 0.25 });
          } else {
            gsap.to(card, { opacity: 0, scale: 0.9, duration: 0.2, onComplete: () => { card.style.display = 'none'; } });
          }
        });
      });
    });

    // Nav link filter
    document.querySelectorAll('.nav-links a[data-filter]').forEach(a => {
      a.addEventListener('click', e => {
        e.preventDefault();
        document.querySelector(`.tab-btn[data-cat="${a.dataset.filter}"]`).click();
        document.getElementById('tools').scrollIntoView({ behavior: 'smooth' });
      });
    });

    // Search filter
    const searchInput = document.getElementById('search-input');
    function doSearch() {
      const q = searchInput.value.toLowerCase().trim();
      tabBtns.forEach(b => b.classList.remove('active'));
      if (!q) { document.querySelector('.tab-btn[data-cat="all"]').classList.add('active'); }
      toolCards.forEach(card => {
        const name = card.querySelector('.tool-name').textContent.toLowerCase();
        const desc = card.querySelector('.tool-desc').textContent.toLowerCase();
        const show = !q || name.includes(q) || desc.includes(q);
        card.style.display = show ? 'flex' : 'none';
        if (show) gsap.to(card, { opacity: 1, scale: 1, duration: 0.2 });
      });
    }
    searchInput.addEventListener('input', doSearch);
    document.getElementById('search-btn').addEventListener('click', doSearch);

    // Scroll-triggered card reveal
    gsap.utils.toArray('.tool-card').forEach((card, i) => {
      gsap.fromTo(card, { opacity: 0, scale: 0.9, y: 20 }, {
        opacity: 1, scale: 1, y: 0, duration: 0.4,
        delay: (i % 4) * 0.08,
        scrollTrigger: { trigger: card, start: 'top 92%' }
      });
    });

    // Stats counter
    document.querySelectorAll('.stat-num').forEach(el => {
      const target = parseFloat(el.dataset.target);
      const suffix = el.dataset.suffix || '';
      ScrollTrigger.create({
        trigger: el, start: 'top 90%', once: true,
        onEnter: () => gsap.to({ val: 0 }, {
          val: target, duration: 1.5, ease: 'power2.out',
          onUpdate() { el.textContent = Math.round(this.targets()[0].val) + suffix; }
        })
      });
    });
  </script>
</body>
</html>
```

- [ ] **Step 2: Open in browser and verify**

Open `http://localhost:3456/index.html`

Expected:
- Nav appears at top (transparent, becomes glass on scroll)
- Hero: badge → headline → subtext → search bar animate in
- 16 tool cards in a 4-column grid (all opacity:0 initially, animate in on scroll/load)
- Category tabs: clicking "🔤 Text (10)" hides image cards, "🖼️ Image (6)" hides text cards
- Search: typing "word" shows only Word Counter card
- Stats numbers count up when scrolled into view
- Footer visible at bottom

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add AIWowTools homepage with search, tabs, and GSAP animations"
```

---

## Task 2: Shared Tool Page Shell + Word Counter

**Context:** All tool pages use the same HTML shell. This task defines it in full while building the first tool. Tasks 3–16 reuse this shell — the orchestrator will provide this complete shell as context when dispatching those tasks.

**Files:**
- Create: `tools/word-counter.html`

- [ ] **Step 1: Create `tools/word-counter.html`**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <title>Word Counter — AIWowTools</title>
  <!-- SHARED HEAD (CDN block from plan header) -->
  <style>
    /* SHARED CSS (full block from plan header) */

    /* TOOL PAGE SPECIFIC */
    .page-wrap { max-width: 860px; margin: 0 auto; padding: 0 24px 80px; }
    .breadcrumb { display: flex; align-items: center; gap: 8px; font-size: 0.8rem; color: var(--muted); padding: 20px 0 28px; }
    .breadcrumb a:hover { color: var(--text); }
    .breadcrumb span { color: var(--border); }
    .tool-header { display: flex; align-items: flex-start; gap: 16px; margin-bottom: 32px; }
    .tool-header-icon { font-size: 2.5rem; }
    .tool-header h1 { font-size: clamp(1.5rem, 3vw, 2.25rem); font-weight: 800; background: var(--gradient); -webkit-background-clip: text; -webkit-text-fill-color: transparent; background-clip: text; line-height: 1.2; margin-bottom: 6px; }
    .tool-header p { font-size: 0.9rem; color: var(--muted); }
    .panel { background: var(--surface); border: 1px solid var(--border); border-radius: 14px; padding: 24px; margin-bottom: 20px; }
    .panel-label { font-size: 0.75rem; font-weight: 600; text-transform: uppercase; letter-spacing: 0.08em; color: var(--muted); margin-bottom: 10px; }
    .btn-row { display: flex; align-items: center; gap: 10px; margin-top: 12px; flex-wrap: wrap; }
    /* WORD COUNTER SPECIFIC */
    .stats-grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 12px; }
    @media (max-width: 600px) { .stats-grid { grid-template-columns: repeat(2, 1fr); } }
    .stat-card { background: var(--bg); border: 1px solid var(--border); border-radius: 10px; padding: 16px; text-align: center; }
    .stat-card .num { font-family: 'Space Grotesk', sans-serif; font-size: 1.75rem; font-weight: 800; background: var(--gradient); -webkit-background-clip: text; -webkit-text-fill-color: transparent; background-clip: text; }
    .stat-card .lbl { font-size: 0.75rem; color: var(--muted); margin-top: 4px; }
    /* RELATED TOOLS */
    .related { margin-top: 60px; }
    .related h2 { font-size: 1.25rem; font-weight: 700; margin-bottom: 20px; }
    .related-grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 14px; }
    @media (max-width: 600px) { .related-grid { grid-template-columns: 1fr; } }
  </style>
</head>
<body>
  <nav>
    <a href="../index.html" class="nav-logo">⚡ AIWow<span>Tools</span></a>
    <div class="nav-links">
      <a href="../index.html" data-filter="text">Text Tools</a>
      <a href="../index.html" data-filter="image">Image Tools</a>
      <a href="../index.html">All Tools</a>
    </div>
    <a href="../index.html#tools" class="nav-cta btn">Browse Tools</a>
  </nav>

  <div class="page-wrap">
    <nav class="breadcrumb" aria-label="breadcrumb">
      <a href="../index.html">⚡ AIWowTools</a>
      <span>›</span>
      <a href="../index.html">Text Tools</a>
      <span>›</span>
      <span style="color:var(--primary-light)">Word Counter</span>
    </nav>

    <div class="tool-header">
      <div class="tool-header-icon">🔤</div>
      <div>
        <span class="tool-badge text" style="margin-bottom:8px;display:inline-block">Text Tool</span>
        <h1>Word Counter</h1>
        <p>Count words, characters, sentences, paragraphs and reading time in real-time.</p>
      </div>
    </div>

    <!-- TOOL INTERFACE -->
    <div class="panel">
      <div class="panel-label">Your Text</div>
      <textarea id="input-text" rows="8" placeholder="Paste or type your text here..."></textarea>
      <div class="btn-row">
        <button class="btn btn-secondary" id="clear-btn">Clear</button>
        <span id="live-word-count" style="font-size:0.8rem;color:var(--muted)">0 words</span>
      </div>
    </div>

    <div class="panel">
      <div class="panel-label">Results</div>
      <div class="stats-grid">
        <div class="stat-card"><div class="num" id="word-count">0</div><div class="lbl">Words</div></div>
        <div class="stat-card"><div class="num" id="char-count">0</div><div class="lbl">Characters</div></div>
        <div class="stat-card"><div class="num" id="char-no-space">0</div><div class="lbl">Chars (no spaces)</div></div>
        <div class="stat-card"><div class="num" id="sentence-count">0</div><div class="lbl">Sentences</div></div>
        <div class="stat-card"><div class="num" id="para-count">0</div><div class="lbl">Paragraphs</div></div>
        <div class="stat-card"><div class="num" id="read-time">0 min</div><div class="lbl">Reading Time</div></div>
      </div>
    </div>

    <!-- RELATED TOOLS -->
    <div class="related">
      <h2>More Text Tools</h2>
      <div class="related-grid">
        <a href="case-converter.html" class="tool-card" data-cat="text" style="opacity:1">
          <div class="tool-icon">🔡</div><span class="tool-badge text">Text</span>
          <div class="tool-name">Case Converter</div>
          <div class="tool-desc">UPPER, lower, Title, camelCase</div>
        </a>
        <a href="lorem-ipsum.html" class="tool-card" data-cat="text" style="opacity:1">
          <div class="tool-icon">📝</div><span class="tool-badge text">Text</span>
          <div class="tool-name">Lorem Ipsum</div>
          <div class="tool-desc">Generate placeholder text</div>
        </a>
        <a href="text-diff.html" class="tool-card" data-cat="text" style="opacity:1">
          <div class="tool-icon">🔀</div><span class="tool-badge text">Text</span>
          <div class="tool-name">Text Diff</div>
          <div class="tool-desc">Compare two texts</div>
        </a>
      </div>
    </div>
  </div>

  <!-- FOOTER -->
  <footer>
    <div class="footer-inner">
      <div>
        <div class="footer-logo">⚡ AIWow<span>Tools</span></div>
        <p class="footer-tagline">The only toolbox you'll ever need.</p>
      </div>
      <div class="footer-col">
        <h4>Tools</h4>
        <a href="../index.html">Text Tools</a>
        <a href="../index.html">Image Tools</a>
        <a href="../index.html#tools">All Tools</a>
      </div>
      <div class="footer-col">
        <h4>Company</h4>
        <a href="#">About</a><a href="#">Privacy</a><a href="#">Contact</a>
      </div>
    </div>
    <p class="footer-copy">© 2026 AIWowTools. Free forever.</p>
  </footer>

  <script>
    // SHARED LENIS + NAV SCROLL JS (paste shared block from plan header)

    // Page load animation
    gsap.from('.tool-header', { opacity: 0, y: 20, duration: 0.5, delay: 0.1 });
    gsap.from('.panel', { opacity: 0, y: 20, duration: 0.4, stagger: 0.1, delay: 0.2 });
    gsap.utils.toArray('.related .tool-card').forEach((card, i) => {
      gsap.from(card, { opacity: 0, scale: 0.9, duration: 0.35, delay: i * 0.08,
        scrollTrigger: { trigger: card, start: 'top 92%' } });
    });

    // Word counter logic
    const ta = document.getElementById('input-text');
    function analyze() {
      const text = ta.value;
      const words = text.trim() === '' ? 0 : text.trim().split(/\s+/).length;
      const chars = text.length;
      const charsNoSpace = text.replace(/\s/g, '').length;
      const sentences = text.trim() === '' ? 0 : (text.match(/[^.!?]*[.!?]/g) || []).length;
      const paras = text.trim() === '' ? 0 : text.split(/\n\s*\n/).filter(p => p.trim()).length || 1;
      const readTime = words === 0 ? 0 : Math.max(1, Math.ceil(words / 200));
      document.getElementById('word-count').textContent = words.toLocaleString();
      document.getElementById('char-count').textContent = chars.toLocaleString();
      document.getElementById('char-no-space').textContent = charsNoSpace.toLocaleString();
      document.getElementById('sentence-count').textContent = sentences.toLocaleString();
      document.getElementById('para-count').textContent = paras.toLocaleString();
      document.getElementById('read-time').textContent = readTime + ' min';
      document.getElementById('live-word-count').textContent = words + ' words';
    }
    ta.addEventListener('input', analyze);
    document.getElementById('clear-btn').addEventListener('click', () => { ta.value = ''; analyze(); });
  </script>
</body>
</html>
```

- [ ] **Step 2: Verify**

Open `http://localhost:3456/tools/word-counter.html`

Expected:
- Nav visible, breadcrumb shows `AIWowTools › Text Tools › Word Counter`
- Tool header with 🔤 icon and gradient title
- Paste "Hello world. This is a test." → stats update in real-time: 6 words, 30 chars, 2 sentences
- Clear button empties textarea and resets all numbers to 0
- Related tools row at bottom, nav links back to `../index.html`

- [ ] **Step 3: Commit**

```bash
git add tools/word-counter.html
git commit -m "feat: add Word Counter tool with real-time word/char/sentence stats"
```

---

## Task 3: Case Converter

**Context:** Use the full HTML shell from Task 2 (nav, breadcrumb, tool-header, panel, related, footer, CSS, GSAP init, Lenis). Change title, breadcrumb, tool-header icon/name/desc, and replace the tool interface + JS.

**Files:**
- Create: `tools/case-converter.html`

- [ ] **Step 1: Create `tools/case-converter.html`**

Use the complete shell from Task 2. Replace the tool interface panel and script with:

**Tool interface HTML** (inside the first `.panel`):
```html
<div class="panel">
  <div class="panel-label">Input Text</div>
  <textarea id="input-text" rows="6" placeholder="Type or paste your text here..."></textarea>
</div>
<div class="panel">
  <div class="panel-label">Convert To</div>
  <div class="btn-row" style="flex-wrap:wrap;gap:10px">
    <button class="btn btn-secondary case-btn" data-mode="upper">UPPERCASE</button>
    <button class="btn btn-secondary case-btn" data-mode="lower">lowercase</button>
    <button class="btn btn-secondary case-btn" data-mode="title">Title Case</button>
    <button class="btn btn-secondary case-btn" data-mode="camel">camelCase</button>
    <button class="btn btn-secondary case-btn" data-mode="snake">snake_case</button>
    <button class="btn btn-secondary case-btn" data-mode="kebab">kebab-case</button>
  </div>
</div>
<div class="panel" id="output-panel" style="display:none">
  <div class="panel-label">Result <span id="mode-label" style="color:var(--primary-light)"></span></div>
  <textarea id="output-text" rows="6" readonly></textarea>
  <div class="btn-row">
    <button class="btn btn-primary" id="copy-btn">Copy Result</button>
    <span class="copy-feedback" id="copy-feedback">✓ Copied!</span>
    <button class="btn btn-secondary" id="clear-btn">Clear</button>
  </div>
</div>
```

**Tool header values:**
- Icon: `🔡`
- Badge: `Text Tool`
- Title: `Case Converter`
- Desc: `Convert text to UPPERCASE, lowercase, Title Case, camelCase, snake_case, or kebab-case.`

**Related tools:** Word Counter, Lorem Ipsum, Text to Slug

**JS logic** (replace script tool section):
```js
function toTitleCase(s) { return s.toLowerCase().replace(/(?:^|\s)\S/g, c => c.toUpperCase()); }
function toCamelCase(s) { return s.toLowerCase().replace(/[^a-zA-Z0-9]+(.)/g, (_,c) => c.toUpperCase()); }
function toSnakeCase(s) { return s.toLowerCase().replace(/[^a-z0-9]+/g, '_').replace(/^_|_$/g,''); }
function toKebabCase(s) { return s.toLowerCase().replace(/[^a-z0-9]+/g, '-').replace(/^-|-$/g,''); }

const modeLabels = { upper:'UPPERCASE', lower:'lowercase', title:'Title Case', camel:'camelCase', snake:'snake_case', kebab:'kebab-case' };

document.querySelectorAll('.case-btn').forEach(btn => {
  btn.addEventListener('click', () => {
    const text = document.getElementById('input-text').value;
    if (!text.trim()) return;
    const mode = btn.dataset.mode;
    let result = '';
    if (mode === 'upper') result = text.toUpperCase();
    else if (mode === 'lower') result = text.toLowerCase();
    else if (mode === 'title') result = toTitleCase(text);
    else if (mode === 'camel') result = toCamelCase(text);
    else if (mode === 'snake') result = toSnakeCase(text);
    else if (mode === 'kebab') result = toKebabCase(text);
    document.getElementById('output-text').value = result;
    document.getElementById('mode-label').textContent = modeLabels[mode];
    document.getElementById('output-panel').style.display = 'block';
  });
});

document.getElementById('copy-btn').addEventListener('click', () => {
  navigator.clipboard.writeText(document.getElementById('output-text').value);
  const fb = document.getElementById('copy-feedback');
  fb.style.opacity = '1';
  setTimeout(() => fb.style.opacity = '0', 2000);
});
document.getElementById('clear-btn').addEventListener('click', () => {
  document.getElementById('input-text').value = '';
  document.getElementById('output-text').value = '';
  document.getElementById('output-panel').style.display = 'none';
});
```

- [ ] **Step 2: Verify**

Open `http://localhost:3456/tools/case-converter.html`

Type "Hello World from AIWowTools" → click each button:
- UPPERCASE → `HELLO WORLD FROM AIWOWTOOLS`
- lowercase → `hello world from aiwowtools`
- Title Case → `Hello World From Aiwowtools`
- camelCase → `helloWorldFromAiwowtools`
- snake_case → `hello_world_from_aiwowtools`
- kebab-case → `hello-world-from-aiwowtools`
- Copy button shows `✓ Copied!` for 2 seconds

- [ ] **Step 3: Commit**

```bash
git add tools/case-converter.html
git commit -m "feat: add Case Converter with 6 conversion modes"
```

---

## Task 4: Lorem Ipsum Generator

**Files:**
- Create: `tools/lorem-ipsum.html`

- [ ] **Step 1: Create `tools/lorem-ipsum.html`**

Use complete shell from Task 2. Tool header: icon `📝`, title `Lorem Ipsum Generator`, desc `Generate placeholder lorem ipsum text by paragraphs, sentences, or words.`

**Tool interface HTML:**
```html
<div class="panel">
  <div class="panel-label">Generate</div>
  <div class="btn-row" style="align-items:center;gap:12px;flex-wrap:wrap">
    <input type="number" id="count-input" value="3" min="1" max="20" style="width:80px">
    <select id="unit-select">
      <option value="paragraphs">Paragraphs</option>
      <option value="sentences">Sentences</option>
      <option value="words">Words</option>
    </select>
    <button class="btn btn-primary" id="generate-btn">Generate</button>
  </div>
</div>
<div class="panel">
  <div class="panel-label">Output</div>
  <textarea id="output-text" rows="10" readonly placeholder="Generated text will appear here..."></textarea>
  <div class="btn-row">
    <button class="btn btn-primary" id="copy-btn">Copy Text</button>
    <span class="copy-feedback" id="copy-feedback">✓ Copied!</span>
  </div>
</div>
```

**Related tools:** Word Counter, Case Converter, Find & Replace

**JS logic:**
```js
const WORDS = ['lorem','ipsum','dolor','sit','amet','consectetur','adipiscing','elit','sed','do','eiusmod','tempor','incididunt','ut','labore','et','dolore','magna','aliqua','enim','ad','minim','veniam','quis','nostrud','exercitation','ullamco','laboris','nisi','aliquip','ex','ea','commodo','consequat','duis','aute','irure','in','reprehenderit','voluptate','velit','esse','cillum','eu','fugiat','nulla','pariatur','excepteur','sint','occaecat','cupidatat','non','proident','sunt','culpa','qui','officia','deserunt','mollit','anim','id','est','laborum'];
const rw = () => WORDS[Math.floor(Math.random() * WORDS.length)];
const rs = () => { const len = 8 + Math.floor(Math.random()*10); const words = Array.from({length:len},rw); words[0] = words[0][0].toUpperCase()+words[0].slice(1); return words.join(' ') + '.'; };
const rp = () => Array.from({length: 4 + Math.floor(Math.random()*3)}, rs).join(' ');

document.getElementById('generate-btn').addEventListener('click', () => {
  const count = Math.min(20, Math.max(1, parseInt(document.getElementById('count-input').value) || 3));
  const unit = document.getElementById('unit-select').value;
  let result = '';
  if (unit === 'paragraphs') result = Array.from({length:count}, rp).join('\n\n');
  else if (unit === 'sentences') result = Array.from({length:count}, rs).join(' ');
  else result = Array.from({length:count}, rw).join(' ');
  document.getElementById('output-text').value = result;
});

document.getElementById('copy-btn').addEventListener('click', () => {
  navigator.clipboard.writeText(document.getElementById('output-text').value);
  const fb = document.getElementById('copy-feedback');
  fb.style.opacity = '1';
  setTimeout(() => fb.style.opacity = '0', 2000);
});
```

- [ ] **Step 2: Verify**

Open `http://localhost:3456/tools/lorem-ipsum.html`
- Set count to 2, unit Paragraphs → click Generate → 2 paragraphs of lorem ipsum appear
- Change unit to Sentences, count 5 → 5 sentences appear
- Copy button copies the text

- [ ] **Step 3: Commit**

```bash
git add tools/lorem-ipsum.html
git commit -m "feat: add Lorem Ipsum Generator with paragraph/sentence/word modes"
```

---

## Task 5: Text Diff

**Files:**
- Create: `tools/text-diff.html`

- [ ] **Step 1: Create `tools/text-diff.html`**

Use complete shell from Task 2. Tool header: icon `🔀`, title `Text Diff`, desc `Compare two texts side by side. Added lines highlighted green, removed lines highlighted red.`

**Add to `<style>` (inside the shared CSS):**
```css
.diff-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 16px; }
@media (max-width: 640px) { .diff-grid { grid-template-columns: 1fr; } }
.diff-output { background: var(--bg); border: 1px solid var(--border); border-radius: 8px; padding: 12px; font-family: 'Courier New', monospace; font-size: 0.82rem; min-height: 120px; max-height: 320px; overflow-y: auto; white-space: pre-wrap; word-break: break-all; }
.diff-add { background: rgba(34,197,94,0.12); color: #86efac; border-left: 3px solid #22c55e; padding-left: 6px; }
.diff-rm  { background: rgba(239,68,68,0.12); color: #fca5a5; border-left: 3px solid #ef4444; padding-left: 6px; }
.diff-eq  { color: var(--muted); padding-left: 6px; }
.diff-summary { font-size: 0.8rem; color: var(--muted); margin-top: 10px; }
```

**Tool interface HTML:**
```html
<div class="panel">
  <div class="diff-grid">
    <div>
      <div class="panel-label">Original Text (A)</div>
      <textarea id="text-a" rows="8" placeholder="Paste original text here..."></textarea>
    </div>
    <div>
      <div class="panel-label">Modified Text (B)</div>
      <textarea id="text-b" rows="8" placeholder="Paste modified text here..."></textarea>
    </div>
  </div>
  <div class="btn-row" style="margin-top:14px">
    <button class="btn btn-primary" id="compare-btn">Compare</button>
    <button class="btn btn-secondary" id="clear-btn">Clear</button>
  </div>
</div>
<div class="panel" id="result-panel" style="display:none">
  <div class="panel-label">Diff Result</div>
  <div class="diff-output" id="diff-output"></div>
  <div class="diff-summary" id="diff-summary"></div>
</div>
```

**Related tools:** Word Counter, Remove Duplicates, Find & Replace

**JS logic:**
```js
function escHtml(s) { return s.replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;'); }
function lcsMatrix(a, b) {
  const m = a.length, n = b.length;
  const dp = Array.from({length:m+1}, () => new Array(n+1).fill(0));
  for (let i=1;i<=m;i++) for (let j=1;j<=n;j++)
    dp[i][j] = a[i-1]===b[j-1] ? dp[i-1][j-1]+1 : Math.max(dp[i-1][j],dp[i][j-1]);
  return dp;
}
function buildDiff(a, b) {
  const dp = lcsMatrix(a, b);
  const diff = [];
  let i = a.length, j = b.length;
  while (i > 0 || j > 0) {
    if (i>0&&j>0&&a[i-1]===b[j-1]) { diff.unshift({t:'eq',s:a[i-1]}); i--;j--; }
    else if (j>0&&(i===0||dp[i][j-1]>=dp[i-1][j])) { diff.unshift({t:'add',s:b[j-1]}); j--; }
    else { diff.unshift({t:'rm',s:a[i-1]}); i--; }
  }
  return diff;
}
document.getElementById('compare-btn').addEventListener('click', () => {
  const a = document.getElementById('text-a').value.split('\n');
  const b = document.getElementById('text-b').value.split('\n');
  const diff = buildDiff(a, b);
  let adds=0, rms=0;
  const html = diff.map(d => {
    if (d.t==='add') { adds++; return `<div class="diff-add">+ ${escHtml(d.s)}</div>`; }
    if (d.t==='rm')  { rms++;  return `<div class="diff-rm">- ${escHtml(d.s)}</div>`; }
    return `<div class="diff-eq">  ${escHtml(d.s)}</div>`;
  }).join('');
  document.getElementById('diff-output').innerHTML = html;
  document.getElementById('diff-summary').textContent = `${adds} line(s) added · ${rms} line(s) removed`;
  document.getElementById('result-panel').style.display = 'block';
});
document.getElementById('clear-btn').addEventListener('click', () => {
  document.getElementById('text-a').value = '';
  document.getElementById('text-b').value = '';
  document.getElementById('result-panel').style.display = 'none';
});
```

- [ ] **Step 2: Verify**

Open `http://localhost:3456/tools/text-diff.html`
- Text A: "hello\nworld\nfoo" | Text B: "hello\nearth\nfoo\nbar" → click Compare
- "world" line → red (removed), "earth" and "bar" → green (added), "hello" and "foo" → muted (unchanged)
- Summary shows "2 line(s) added · 1 line(s) removed"

- [ ] **Step 3: Commit**

```bash
git add tools/text-diff.html
git commit -m "feat: add Text Diff tool with LCS-based line comparison"
```

---

## Task 6: Remove Duplicates

**Files:**
- Create: `tools/remove-duplicates.html`

- [ ] **Step 1: Create `tools/remove-duplicates.html`**

Tool header: icon `🧹`, title `Remove Duplicates`, desc `Remove duplicate lines from any text block. Case-sensitive option included.`

**Tool interface HTML:**
```html
<div class="panel">
  <div class="panel-label">Input Text (one item per line)</div>
  <textarea id="input-text" rows="8" placeholder="apple&#10;banana&#10;apple&#10;cherry&#10;banana"></textarea>
  <div class="btn-row" style="margin-top:12px;flex-wrap:wrap;gap:14px">
    <label style="display:flex;align-items:center;gap:6px;font-size:0.85rem;cursor:pointer">
      <input type="checkbox" id="case-sensitive" style="width:auto;accent-color:var(--primary)"> Case-sensitive
    </label>
    <label style="display:flex;align-items:center;gap:6px;font-size:0.85rem;cursor:pointer">
      <input type="checkbox" id="trim-lines" checked style="width:auto;accent-color:var(--primary)"> Trim whitespace
    </label>
    <button class="btn btn-primary" id="remove-btn">Remove Duplicates</button>
    <button class="btn btn-secondary" id="clear-btn">Clear</button>
  </div>
</div>
<div class="panel" id="output-panel" style="display:none">
  <div class="panel-label">Result <span id="removed-count" style="color:var(--primary-light);font-size:0.8rem"></span></div>
  <textarea id="output-text" rows="8" readonly></textarea>
  <div class="btn-row">
    <button class="btn btn-primary" id="copy-btn">Copy Result</button>
    <span class="copy-feedback" id="copy-feedback">✓ Copied!</span>
  </div>
</div>
```

**Related tools:** Line Sorter, Whitespace Cleaner, Find & Replace

**JS logic:**
```js
document.getElementById('remove-btn').addEventListener('click', () => {
  const input = document.getElementById('input-text').value;
  const caseSensitive = document.getElementById('case-sensitive').checked;
  const trimLines = document.getElementById('trim-lines').checked;
  let lines = input.split('\n');
  if (trimLines) lines = lines.map(l => l.trim());
  const seen = new Set();
  const result = lines.filter(line => {
    const key = caseSensitive ? line : line.toLowerCase();
    if (seen.has(key)) return false;
    seen.add(key); return true;
  });
  const removed = lines.length - result.length;
  document.getElementById('output-text').value = result.join('\n');
  document.getElementById('removed-count').textContent = `— ${removed} duplicate${removed!==1?'s':''} removed`;
  document.getElementById('output-panel').style.display = 'block';
});
document.getElementById('copy-btn').addEventListener('click', () => {
  navigator.clipboard.writeText(document.getElementById('output-text').value);
  const fb = document.getElementById('copy-feedback');
  fb.style.opacity='1'; setTimeout(()=>fb.style.opacity='0',2000);
});
document.getElementById('clear-btn').addEventListener('click', () => {
  document.getElementById('input-text').value = '';
  document.getElementById('output-panel').style.display = 'none';
});
```

- [ ] **Step 2: Verify**

Open `http://localhost:3456/tools/remove-duplicates.html`
- Input: "apple\nbanana\napple\ncherry\nbanana" → Remove Duplicates
- Output: "apple\nbanana\ncherry" — summary shows "2 duplicates removed"
- With case-sensitive unchecked: "Apple" and "apple" are treated as same

- [ ] **Step 3: Commit**

```bash
git add tools/remove-duplicates.html
git commit -m "feat: add Remove Duplicates tool with case-sensitivity option"
```

---

## Task 7: Text Reverser

**Files:**
- Create: `tools/text-reverser.html`

- [ ] **Step 1: Create `tools/text-reverser.html`**

Tool header: icon `🔄`, title `Text Reverser`, desc `Reverse all characters or just the word order in your text.`

**Tool interface HTML:**
```html
<div class="panel">
  <div class="panel-label">Input Text</div>
  <textarea id="input-text" rows="6" placeholder="Type or paste text to reverse..."></textarea>
  <div class="btn-row" style="margin-top:12px">
    <button class="btn btn-primary" id="reverse-all-btn">Reverse Characters</button>
    <button class="btn btn-secondary" id="reverse-words-btn">Reverse Word Order</button>
    <button class="btn btn-secondary" id="clear-btn">Clear</button>
  </div>
</div>
<div class="panel" id="output-panel" style="display:none">
  <div class="panel-label">Result <span id="mode-label" style="color:var(--primary-light);font-size:0.8rem"></span></div>
  <textarea id="output-text" rows="6" readonly></textarea>
  <div class="btn-row">
    <button class="btn btn-primary" id="copy-btn">Copy</button>
    <span class="copy-feedback" id="copy-feedback">✓ Copied!</span>
  </div>
</div>
```

**Related tools:** Case Converter, Whitespace Cleaner, Text to Slug

**JS logic:**
```js
function showResult(text, label) {
  document.getElementById('output-text').value = text;
  document.getElementById('mode-label').textContent = '— ' + label;
  document.getElementById('output-panel').style.display = 'block';
}
document.getElementById('reverse-all-btn').addEventListener('click', () => {
  showResult(document.getElementById('input-text').value.split('').reverse().join(''), 'Characters reversed');
});
document.getElementById('reverse-words-btn').addEventListener('click', () => {
  showResult(document.getElementById('input-text').value.trim().split(/\s+/).reverse().join(' '), 'Word order reversed');
});
document.getElementById('copy-btn').addEventListener('click', () => {
  navigator.clipboard.writeText(document.getElementById('output-text').value);
  const fb = document.getElementById('copy-feedback');
  fb.style.opacity='1'; setTimeout(()=>fb.style.opacity='0',2000);
});
document.getElementById('clear-btn').addEventListener('click', () => {
  document.getElementById('input-text').value = '';
  document.getElementById('output-panel').style.display = 'none';
});
```

- [ ] **Step 2: Verify**

Open `http://localhost:3456/tools/text-reverser.html`
- Input: "Hello World" → Reverse Characters → `dlroW olleH`
- Input: "Hello World" → Reverse Word Order → `World Hello`

- [ ] **Step 3: Commit**

```bash
git add tools/text-reverser.html
git commit -m "feat: add Text Reverser with character and word-order modes"
```

---

## Task 8: Whitespace Cleaner

**Files:**
- Create: `tools/whitespace-cleaner.html`

- [ ] **Step 1: Create `tools/whitespace-cleaner.html`**

Tool header: icon `✂️`, title `Whitespace Cleaner`, desc `Trim leading/trailing spaces, normalize multiple spaces, and remove blank lines.`

**Tool interface HTML:**
```html
<div class="panel">
  <div class="panel-label">Options</div>
  <div style="display:flex;flex-direction:column;gap:10px">
    <label style="display:flex;align-items:center;gap:8px;font-size:0.875rem;cursor:pointer">
      <input type="checkbox" id="trim-ends" checked style="width:auto;accent-color:var(--primary)">
      Trim leading &amp; trailing spaces on each line
    </label>
    <label style="display:flex;align-items:center;gap:8px;font-size:0.875rem;cursor:pointer">
      <input type="checkbox" id="normalize-spaces" checked style="width:auto;accent-color:var(--primary)">
      Normalize multiple spaces to single space
    </label>
    <label style="display:flex;align-items:center;gap:8px;font-size:0.875rem;cursor:pointer">
      <input type="checkbox" id="remove-blank" style="width:auto;accent-color:var(--primary)">
      Remove blank lines
    </label>
  </div>
</div>
<div class="panel">
  <div class="panel-label">Input Text</div>
  <textarea id="input-text" rows="7" placeholder="Paste text with messy whitespace here..."></textarea>
</div>
<div class="panel">
  <div class="panel-label">Cleaned Result</div>
  <textarea id="output-text" rows="7" readonly></textarea>
  <div class="btn-row">
    <button class="btn btn-primary" id="copy-btn">Copy Result</button>
    <span class="copy-feedback" id="copy-feedback">✓ Copied!</span>
  </div>
</div>
```

**Related tools:** Remove Duplicates, Find & Replace, Text to Slug

**JS logic:**
```js
function clean() {
  let text = document.getElementById('input-text').value;
  if (document.getElementById('trim-ends').checked)
    text = text.split('\n').map(l => l.trim()).join('\n');
  if (document.getElementById('normalize-spaces').checked)
    text = text.replace(/ {2,}/g, ' ');
  if (document.getElementById('remove-blank').checked)
    text = text.replace(/^\s*\n/gm, '').replace(/\n{2,}/g, '\n');
  document.getElementById('output-text').value = text;
}
document.getElementById('input-text').addEventListener('input', clean);
document.querySelectorAll('input[type="checkbox"]').forEach(cb => cb.addEventListener('change', clean));
document.getElementById('copy-btn').addEventListener('click', () => {
  navigator.clipboard.writeText(document.getElementById('output-text').value);
  const fb = document.getElementById('copy-feedback');
  fb.style.opacity='1'; setTimeout(()=>fb.style.opacity='0',2000);
});
```

- [ ] **Step 2: Verify**

Open `http://localhost:3456/tools/whitespace-cleaner.html`
- Input: `"  hello   world  "` → output `"hello world"` (trim + normalize)
- Enable "Remove blank lines" → blank lines disappear from output in real-time

- [ ] **Step 3: Commit**

```bash
git add tools/whitespace-cleaner.html
git commit -m "feat: add Whitespace Cleaner with real-time trim/normalize/blank-line options"
```

---

## Task 9: Text to Slug

**Files:**
- Create: `tools/text-to-slug.html`

- [ ] **Step 1: Create `tools/text-to-slug.html`**

Tool header: icon `🔗`, title `Text to Slug`, desc `Convert any text to a URL-friendly slug. Removes special characters and spaces.`

**Tool interface HTML:**
```html
<div class="panel">
  <div class="panel-label">Input Text</div>
  <input type="text" id="input-text" placeholder="e.g. My Blog Post Title (2026)!">
</div>
<div class="panel">
  <div class="panel-label">Generated Slug</div>
  <div style="display:flex;gap:10px;align-items:center">
    <input type="text" id="output-text" readonly style="flex:1;color:var(--primary-light);font-family:'Courier New',monospace">
    <button class="btn btn-primary" id="copy-btn">Copy</button>
    <span class="copy-feedback" id="copy-feedback">✓ Copied!</span>
  </div>
  <p id="slug-preview" style="font-size:0.78rem;color:var(--muted);margin-top:10px"></p>
</div>
```

**Related tools:** Case Converter, Whitespace Cleaner, Find & Replace

**JS logic:**
```js
function toSlug(str) {
  return str.toLowerCase()
    .normalize('NFD').replace(/[̀-ͯ]/g,'')
    .replace(/[^a-z0-9\s-]/g,'')
    .trim().replace(/\s+/g,'-')
    .replace(/-+/g,'-').replace(/^-|-$/g,'');
}
document.getElementById('input-text').addEventListener('input', function() {
  const slug = toSlug(this.value);
  document.getElementById('output-text').value = slug;
  document.getElementById('slug-preview').textContent = slug ? 'Preview: https://yoursite.com/' + slug : '';
});
document.getElementById('copy-btn').addEventListener('click', () => {
  navigator.clipboard.writeText(document.getElementById('output-text').value);
  const fb = document.getElementById('copy-feedback');
  fb.style.opacity='1'; setTimeout(()=>fb.style.opacity='0',2000);
});
```

- [ ] **Step 2: Verify**

Open `http://localhost:3456/tools/text-to-slug.html`
- Type "My Blog Post Title (2026)!" → output: `my-blog-post-title-2026`
- Preview shows: `https://yoursite.com/my-blog-post-title-2026`

- [ ] **Step 3: Commit**

```bash
git add tools/text-to-slug.html
git commit -m "feat: add Text to Slug with real-time URL preview"
```

---

## Task 10: Line Sorter

**Files:**
- Create: `tools/line-sorter.html`

- [ ] **Step 1: Create `tools/line-sorter.html`**

Tool header: icon `📋`, title `Line Sorter`, desc `Sort lines alphabetically, by length, or shuffle them randomly.`

**Tool interface HTML:**
```html
<div class="panel">
  <div class="panel-label">Input Lines</div>
  <textarea id="input-text" rows="8" placeholder="banana&#10;apple&#10;cherry&#10;date&#10;elderberry"></textarea>
  <div class="btn-row" style="margin-top:12px;flex-wrap:wrap">
    <button class="btn btn-primary" id="sort-az">A → Z</button>
    <button class="btn btn-secondary" id="sort-za">Z → A</button>
    <button class="btn btn-secondary" id="sort-len-asc">Shortest first</button>
    <button class="btn btn-secondary" id="sort-len-desc">Longest first</button>
    <button class="btn btn-secondary" id="shuffle">🔀 Shuffle</button>
    <button class="btn btn-secondary" id="clear-btn">Clear</button>
  </div>
</div>
<div class="panel" id="output-panel" style="display:none">
  <div class="panel-label">Sorted Result <span id="sort-label" style="color:var(--primary-light);font-size:0.8rem"></span></div>
  <textarea id="output-text" rows="8" readonly></textarea>
  <div class="btn-row">
    <button class="btn btn-primary" id="copy-btn">Copy</button>
    <span class="copy-feedback" id="copy-feedback">✓ Copied!</span>
  </div>
</div>
```

**Related tools:** Remove Duplicates, Find & Replace, Whitespace Cleaner

**JS logic:**
```js
function sort(fn, label) {
  const lines = document.getElementById('input-text').value.split('\n');
  document.getElementById('output-text').value = fn(lines).join('\n');
  document.getElementById('sort-label').textContent = '— ' + label;
  document.getElementById('output-panel').style.display = 'block';
}
document.getElementById('sort-az').addEventListener('click', () => sort(l => [...l].sort((a,b)=>a.localeCompare(b)), 'A → Z'));
document.getElementById('sort-za').addEventListener('click', () => sort(l => [...l].sort((a,b)=>b.localeCompare(a)), 'Z → A'));
document.getElementById('sort-len-asc').addEventListener('click', () => sort(l => [...l].sort((a,b)=>a.length-b.length), 'Shortest first'));
document.getElementById('sort-len-desc').addEventListener('click', () => sort(l => [...l].sort((a,b)=>b.length-a.length), 'Longest first'));
document.getElementById('shuffle').addEventListener('click', () => sort(l => { const a=[...l]; for(let i=a.length-1;i>0;i--){const j=Math.floor(Math.random()*(i+1));[a[i],a[j]]=[a[j],a[i]];} return a; }, 'Shuffled'));
document.getElementById('copy-btn').addEventListener('click', () => {
  navigator.clipboard.writeText(document.getElementById('output-text').value);
  const fb=document.getElementById('copy-feedback'); fb.style.opacity='1'; setTimeout(()=>fb.style.opacity='0',2000);
});
document.getElementById('clear-btn').addEventListener('click', () => {
  document.getElementById('input-text').value = '';
  document.getElementById('output-panel').style.display = 'none';
});
```

- [ ] **Step 2: Verify**

Open `http://localhost:3456/tools/line-sorter.html`
- Input: "banana\napple\ncherry" → A→Z → "apple\nbanana\ncherry"
- Shuffle → random order each time

- [ ] **Step 3: Commit**

```bash
git add tools/line-sorter.html
git commit -m "feat: add Line Sorter with A-Z, Z-A, length, and shuffle modes"
```

---

## Task 11: Find & Replace

**Files:**
- Create: `tools/find-replace.html`

- [ ] **Step 1: Create `tools/find-replace.html`**

Tool header: icon `🔍`, title `Find & Replace`, desc `Bulk find and replace any word or phrase in your text. Shows match count.`

**Tool interface HTML:**
```html
<div class="panel">
  <div class="panel-label">Input Text</div>
  <textarea id="input-text" rows="7" placeholder="Paste your text here..."></textarea>
</div>
<div class="panel">
  <div class="panel-label">Find &amp; Replace</div>
  <div style="display:grid;grid-template-columns:1fr 1fr;gap:12px;margin-bottom:12px">
    <div>
      <div style="font-size:0.78rem;color:var(--muted);margin-bottom:4px">Find</div>
      <input type="text" id="find-input" placeholder="Text to find...">
    </div>
    <div>
      <div style="font-size:0.78rem;color:var(--muted);margin-bottom:4px">Replace with</div>
      <input type="text" id="replace-input" placeholder="Replace with... (leave blank to delete)">
    </div>
  </div>
  <div class="btn-row">
    <button class="btn btn-primary" id="replace-btn">Replace All</button>
    <button class="btn btn-secondary" id="clear-btn">Clear</button>
    <span id="match-count" style="font-size:0.8rem;color:var(--muted)"></span>
  </div>
</div>
<div class="panel" id="output-panel" style="display:none">
  <div class="panel-label">Result</div>
  <textarea id="output-text" rows="7" readonly></textarea>
  <div class="btn-row">
    <button class="btn btn-primary" id="copy-btn">Copy</button>
    <span class="copy-feedback" id="copy-feedback">✓ Copied!</span>
  </div>
</div>
```

**Related tools:** Whitespace Cleaner, Remove Duplicates, Text Diff

**JS logic:**
```js
document.getElementById('replace-btn').addEventListener('click', () => {
  const text = document.getElementById('input-text').value;
  const find = document.getElementById('find-input').value;
  const replace = document.getElementById('replace-input').value;
  if (!find) { document.getElementById('match-count').textContent = 'Enter a search term.'; return; }
  const escaped = find.replace(/[.*+?^${}()|[\]\\]/g,'\\$&');
  const regex = new RegExp(escaped, 'g');
  const matches = (text.match(regex) || []).length;
  document.getElementById('output-text').value = text.replace(regex, replace);
  document.getElementById('match-count').textContent = matches > 0 ? `${matches} match${matches!==1?'es':''} replaced` : 'No matches found';
  document.getElementById('output-panel').style.display = 'block';
});
document.getElementById('copy-btn').addEventListener('click', () => {
  navigator.clipboard.writeText(document.getElementById('output-text').value);
  const fb=document.getElementById('copy-feedback'); fb.style.opacity='1'; setTimeout(()=>fb.style.opacity='0',2000);
});
document.getElementById('clear-btn').addEventListener('click', () => {
  ['input-text','find-input','replace-input'].forEach(id=>document.getElementById(id).value='');
  document.getElementById('output-panel').style.display='none';
  document.getElementById('match-count').textContent='';
});
```

- [ ] **Step 2: Verify**

Open `http://localhost:3456/tools/find-replace.html`
- Input: "the cat sat on the mat" | Find: "the" | Replace: "a" → "a cat sat on a mat"
- Match count shows "2 matches replaced"
- Find: "xyz" → "No matches found"

- [ ] **Step 3: Commit**

```bash
git add tools/find-replace.html
git commit -m "feat: add Find & Replace with match count"
```

---

## Task 12: Image Resizer

**Context:** All 6 image tools use a shared pattern: drag-drop zone → settings → canvas preview → download. The `setupDropZone`, `loadImage`, and `downloadCanvas` helpers below are defined in every image tool page.

**Files:**
- Create: `tools/image-resizer.html`

- [ ] **Step 1: Create `tools/image-resizer.html`**

Use the Task 2 shell. Tool header: icon `🖼️`, title `Image Resizer`, desc `Resize any image to a custom width and height. Lock aspect ratio to avoid distortion.` Category badge: `Image Tool` (purple).

**Add to `<style>`:**
```css
.drop-zone { background: var(--surface); border: 2px dashed var(--border); border-radius: 14px; padding: 40px 24px; text-align: center; cursor: pointer; transition: border-color 0.2s, background 0.2s; }
.drop-zone:hover, .drop-zone.drag-over { border-color: var(--primary); background: rgba(99,102,241,0.05); }
.drop-zone .drop-icon { font-size: 2.5rem; margin-bottom: 10px; }
.drop-zone p { font-size: 0.875rem; color: var(--muted); }
.drop-zone small { font-size: 0.75rem; color: var(--border); }
#tool-controls { display: none; }
canvas { max-width: 100%; border-radius: 8px; border: 1px solid var(--border); display: block; margin: 0 auto; }
.size-info { font-size: 0.8rem; color: var(--muted); text-align: center; margin-top: 8px; }
.input-row { display: flex; gap: 12px; align-items: flex-end; flex-wrap: wrap; }
.input-group { display: flex; flex-direction: column; gap: 4px; }
.input-group label { font-size: 0.78rem; color: var(--muted); }
.input-group input { width: 100px; }
.lock-label { display: flex; align-items: center; gap: 6px; font-size: 0.85rem; cursor: pointer; padding-top: 20px; }
```

**Tool interface HTML:**
```html
<div class="panel">
  <div class="drop-zone" id="drop-zone">
    <div class="drop-icon">📁</div>
    <p>Drop image here or click to upload</p>
    <small>PNG, JPG, WebP — up to 10MB</small>
    <input type="file" id="file-input" accept="image/*" style="display:none">
  </div>
</div>
<div id="tool-controls">
  <div class="panel">
    <div class="panel-label">Resize Settings</div>
    <div class="input-row">
      <div class="input-group"><label>Width (px)</label><input type="number" id="width-input" min="1"></div>
      <div class="input-group"><label>Height (px)</label><input type="number" id="height-input" min="1"></div>
      <label class="lock-label"><input type="checkbox" id="lock-aspect" checked style="width:auto;accent-color:var(--primary)"> Lock aspect ratio</label>
    </div>
    <p id="orig-size" style="font-size:0.78rem;color:var(--muted);margin-top:10px"></p>
  </div>
  <div class="panel">
    <div class="panel-label">Preview</div>
    <canvas id="preview-canvas"></canvas>
    <div class="btn-row" style="justify-content:center;margin-top:16px">
      <button class="btn btn-primary" id="download-btn">Download Resized Image</button>
    </div>
  </div>
</div>
```

**Related tools:** Image Compressor, Format Converter, Grayscale Converter

**Full JS:**
```js
// SHARED IMAGE UTILS (copy into every image tool page)
function setupDropZone(dropZone, fileInput, onLoad) {
  dropZone.addEventListener('click', () => fileInput.click());
  dropZone.addEventListener('dragover', e => { e.preventDefault(); dropZone.classList.add('drag-over'); });
  dropZone.addEventListener('dragleave', () => dropZone.classList.remove('drag-over'));
  dropZone.addEventListener('drop', e => { e.preventDefault(); dropZone.classList.remove('drag-over'); const f=e.dataTransfer.files[0]; if(f&&f.type.startsWith('image/')) loadImg(f,onLoad); });
  fileInput.addEventListener('change', e => { const f=e.target.files[0]; if(f) loadImg(f,onLoad); });
}
function loadImg(file, cb) {
  const reader = new FileReader();
  reader.onload = e => { const img=new Image(); img.onload=()=>cb(img,file); img.src=e.target.result; };
  reader.readAsDataURL(file);
}
function dlCanvas(canvas, filename) {
  canvas.toBlob(blob => { const a=document.createElement('a'); a.href=URL.createObjectURL(blob); a.download=filename; a.click(); URL.revokeObjectURL(a.href); });
}

// IMAGE RESIZER LOGIC
let currentImg = null;
const canvas = document.getElementById('preview-canvas');
const ctx = canvas.getContext('2d');
const widthInput = document.getElementById('width-input');
const heightInput = document.getElementById('height-input');

setupDropZone(document.getElementById('drop-zone'), document.getElementById('file-input'), (img, file) => {
  currentImg = img;
  widthInput.value = img.naturalWidth;
  heightInput.value = img.naturalHeight;
  document.getElementById('orig-size').textContent = `Original: ${img.naturalWidth}×${img.naturalHeight}px · ${(file.size/1024).toFixed(1)} KB`;
  renderPreview();
  document.getElementById('tool-controls').style.display = 'block';
});

let lockAspect = true;
document.getElementById('lock-aspect').addEventListener('change', e => { lockAspect = e.target.checked; });
widthInput.addEventListener('input', () => {
  if (lockAspect && currentImg) heightInput.value = Math.round(parseInt(widthInput.value) * currentImg.naturalHeight / currentImg.naturalWidth);
  renderPreview();
});
heightInput.addEventListener('input', () => {
  if (lockAspect && currentImg) widthInput.value = Math.round(parseInt(heightInput.value) * currentImg.naturalWidth / currentImg.naturalHeight);
  renderPreview();
});

function renderPreview() {
  if (!currentImg) return;
  const w = parseInt(widthInput.value) || currentImg.naturalWidth;
  const h = parseInt(heightInput.value) || currentImg.naturalHeight;
  canvas.width = w; canvas.height = h;
  ctx.drawImage(currentImg, 0, 0, w, h);
}
document.getElementById('download-btn').addEventListener('click', () => { if(currentImg) dlCanvas(canvas, 'resized.png'); });
```

- [ ] **Step 2: Verify**

Open `http://localhost:3456/tools/image-resizer.html`
- Drop a JPG onto the drop zone → original dimensions appear in settings
- Change width to 200 (with lock on) → height auto-adjusts proportionally
- Canvas preview updates → click Download → file downloads as `resized.png`

- [ ] **Step 3: Commit**

```bash
git add tools/image-resizer.html
git commit -m "feat: add Image Resizer with aspect ratio lock and canvas preview"
```

---

## Task 13: Image Compressor

**Files:**
- Create: `tools/image-compressor.html`

- [ ] **Step 1: Create `tools/image-compressor.html`**

Tool header: icon `🗜️`, title `Image Compressor`, desc `Reduce image file size using quality control. See before/after size comparison.` Category badge: `Image Tool` (purple).

Use the exact same `<style>` additions from Task 12. Include `setupDropZone`, `loadImg`, `dlCanvas` helpers.

**Tool interface HTML:**
```html
<div class="panel">
  <div class="drop-zone" id="drop-zone">
    <div class="drop-icon">📁</div>
    <p>Drop image here or click to upload</p>
    <small>PNG, JPG, WebP — output will be JPEG</small>
    <input type="file" id="file-input" accept="image/*" style="display:none">
  </div>
</div>
<div id="tool-controls">
  <div class="panel">
    <div class="panel-label">Quality</div>
    <div style="display:flex;align-items:center;gap:14px">
      <input type="range" id="quality-slider" min="1" max="100" value="80" style="flex:1;accent-color:var(--primary)">
      <span id="quality-display" style="font-family:'Space Grotesk',sans-serif;font-weight:700;font-size:1.1rem;color:var(--primary-light);min-width:42px">80%</span>
    </div>
    <div style="display:flex;justify-content:space-between;margin-top:14px;font-size:0.85rem;">
      <span>Original: <strong id="orig-size" style="color:var(--text)">—</strong></span>
      <span>Compressed: <strong id="new-size" style="color:var(--primary-light)">—</strong></span>
      <span id="saving" style="color:#34d399"></span>
    </div>
  </div>
  <div class="panel">
    <div class="panel-label">Preview</div>
    <canvas id="preview-canvas"></canvas>
    <div class="btn-row" style="justify-content:center;margin-top:16px">
      <button class="btn btn-primary" id="download-btn">Download Compressed JPEG</button>
    </div>
  </div>
</div>
```

**Related tools:** Image Resizer, Format Converter, Grayscale Converter

**JS logic (tool-specific only — include shared utils from Task 12):**
```js
let currentImg = null, currentFile = null;
const canvas = document.getElementById('preview-canvas');
const ctx = canvas.getContext('2d');
const qualitySlider = document.getElementById('quality-slider');

setupDropZone(document.getElementById('drop-zone'), document.getElementById('file-input'), (img, file) => {
  currentImg = img; currentFile = file;
  canvas.width = img.naturalWidth; canvas.height = img.naturalHeight;
  ctx.drawImage(img, 0, 0);
  document.getElementById('orig-size').textContent = (file.size/1024).toFixed(1)+' KB';
  updateStats();
  document.getElementById('tool-controls').style.display = 'block';
});

qualitySlider.addEventListener('input', () => {
  document.getElementById('quality-display').textContent = qualitySlider.value + '%';
  if (currentImg) updateStats();
});

function updateStats() {
  canvas.toBlob(blob => {
    document.getElementById('new-size').textContent = (blob.size/1024).toFixed(1)+' KB';
    const saving = ((1 - blob.size/currentFile.size)*100).toFixed(0);
    document.getElementById('saving').textContent = saving > 0 ? saving+'% smaller' : '';
  }, 'image/jpeg', parseInt(qualitySlider.value)/100);
}

document.getElementById('download-btn').addEventListener('click', () => {
  if (!currentImg) return;
  canvas.toBlob(blob => { const a=document.createElement('a'); a.href=URL.createObjectURL(blob); a.download='compressed.jpg'; a.click(); URL.revokeObjectURL(a.href); }, 'image/jpeg', parseInt(qualitySlider.value)/100);
});
```

- [ ] **Step 2: Verify**

Open `http://localhost:3456/tools/image-compressor.html`
- Upload a large JPG → "Original: XXX KB" shown
- Drag quality slider to 30% → "Compressed: YYY KB" updates, "Z% smaller" shown
- Download → file saves as `compressed.jpg`

- [ ] **Step 3: Commit**

```bash
git add tools/image-compressor.html
git commit -m "feat: add Image Compressor with quality slider and before/after size stats"
```

---

## Task 14: Format Converter

**Files:**
- Create: `tools/format-converter.html`

- [ ] **Step 1: Create `tools/format-converter.html`**

Tool header: icon `🎨`, title `Format Converter`, desc `Convert images between PNG, JPG, and WebP formats instantly in your browser.`

Include shared CSS (`drop-zone`, `canvas`) and shared utils (`setupDropZone`, `loadImg`, `dlCanvas`) from Task 12.

**Tool interface HTML:**
```html
<div class="panel">
  <div class="drop-zone" id="drop-zone">
    <div class="drop-icon">📁</div>
    <p>Drop image here or click to upload</p>
    <small>PNG, JPG, or WebP accepted</small>
    <input type="file" id="file-input" accept="image/*" style="display:none">
  </div>
</div>
<div id="tool-controls">
  <div class="panel">
    <div class="panel-label">Convert To</div>
    <div class="btn-row">
      <button class="btn btn-primary fmt-btn" data-fmt="png">PNG</button>
      <button class="btn btn-secondary fmt-btn" data-fmt="jpg">JPEG</button>
      <button class="btn btn-secondary fmt-btn" data-fmt="webp">WebP</button>
    </div>
    <p id="format-note" style="font-size:0.78rem;color:var(--muted);margin-top:10px">Select target format then click to download.</p>
  </div>
  <div class="panel">
    <div class="panel-label">Preview</div>
    <canvas id="preview-canvas"></canvas>
    <div class="btn-row" style="justify-content:center;margin-top:16px">
      <button class="btn btn-primary" id="download-btn">Download Converted Image</button>
    </div>
  </div>
</div>
```

**Related tools:** Image Resizer, Image Compressor, Image to Base64

**JS logic:**
```js
let currentImg = null, selectedFmt = 'png';
const canvas = document.getElementById('preview-canvas');
const ctx = canvas.getContext('2d');
const mimeMap = { png:'image/png', jpg:'image/jpeg', webp:'image/webp' };

setupDropZone(document.getElementById('drop-zone'), document.getElementById('file-input'), (img) => {
  currentImg = img;
  canvas.width = img.naturalWidth; canvas.height = img.naturalHeight;
  ctx.drawImage(img, 0, 0);
  document.getElementById('tool-controls').style.display = 'block';
});

document.querySelectorAll('.fmt-btn').forEach(btn => {
  btn.addEventListener('click', () => {
    document.querySelectorAll('.fmt-btn').forEach(b => b.className = 'btn btn-secondary fmt-btn');
    btn.className = 'btn btn-primary fmt-btn';
    selectedFmt = btn.dataset.fmt;
    document.getElementById('format-note').textContent = `Will download as .${selectedFmt}`;
  });
});

document.getElementById('download-btn').addEventListener('click', () => {
  if (!currentImg) return;
  const mime = mimeMap[selectedFmt];
  const quality = selectedFmt === 'jpg' ? 0.92 : undefined;
  canvas.toBlob(blob => { const a=document.createElement('a'); a.href=URL.createObjectURL(blob); a.download=`converted.${selectedFmt}`; a.click(); URL.revokeObjectURL(a.href); }, mime, quality);
});
```

- [ ] **Step 2: Verify**

Open `http://localhost:3456/tools/format-converter.html`
- Upload a PNG → select JPG → Download → file saves as `converted.jpg`
- Select WebP → Download → file saves as `converted.webp`

- [ ] **Step 3: Commit**

```bash
git add tools/format-converter.html
git commit -m "feat: add Format Converter for PNG/JPG/WebP conversion"
```

---

## Task 15: Image to Base64

**Files:**
- Create: `tools/image-to-base64.html`

- [ ] **Step 1: Create `tools/image-to-base64.html`**

Tool header: icon `💾`, title `Image to Base64`, desc `Encode any image to a Base64 string for use in HTML/CSS. Decode Base64 back to an image preview.`

Include shared `drop-zone` CSS and `setupDropZone`, `loadImg` utils from Task 12.

**Tool interface HTML:**
```html
<div class="panel">
  <div class="panel-label">Encode Image → Base64</div>
  <div class="drop-zone" id="drop-zone">
    <div class="drop-icon">📁</div>
    <p>Drop image here or click to upload</p>
    <input type="file" id="file-input" accept="image/*" style="display:none">
  </div>
</div>
<div id="tool-controls">
  <div class="panel">
    <div class="panel-label">Base64 String</div>
    <textarea id="output-text" rows="5" readonly style="font-family:'Courier New',monospace;font-size:0.75rem;word-break:break-all"></textarea>
    <div class="btn-row">
      <button class="btn btn-primary" id="copy-btn">Copy Base64</button>
      <span class="copy-feedback" id="copy-feedback">✓ Copied!</span>
      <span id="base64-size" style="font-size:0.78rem;color:var(--muted)"></span>
    </div>
  </div>
</div>
<div class="panel">
  <div class="panel-label">Decode Base64 → Image</div>
  <textarea id="decode-input" rows="4" placeholder="Paste a Base64 data URL here (data:image/...)"></textarea>
  <div class="btn-row" style="margin-top:10px">
    <button class="btn btn-primary" id="decode-btn">Preview Image</button>
  </div>
  <img id="decoded-preview" style="display:none;max-width:100%;border-radius:8px;margin-top:12px;border:1px solid var(--border)" alt="Decoded preview">
</div>
```

**Related tools:** Format Converter, Image Resizer, Image Compressor

**JS logic:**
```js
setupDropZone(document.getElementById('drop-zone'), document.getElementById('file-input'), (img, file) => {
  const reader = new FileReader();
  reader.onload = e => {
    const b64 = e.target.result;
    document.getElementById('output-text').value = b64;
    document.getElementById('base64-size').textContent = `${(b64.length/1024).toFixed(1)} KB string`;
    document.getElementById('tool-controls').style.display = 'block';
  };
  reader.readAsDataURL(file);
});

document.getElementById('copy-btn').addEventListener('click', () => {
  navigator.clipboard.writeText(document.getElementById('output-text').value);
  const fb=document.getElementById('copy-feedback'); fb.style.opacity='1'; setTimeout(()=>fb.style.opacity='0',2000);
});

document.getElementById('decode-btn').addEventListener('click', () => {
  const b64 = document.getElementById('decode-input').value.trim();
  if (!b64.startsWith('data:image/')) { alert('Please paste a valid Base64 data URL (starts with data:image/)'); return; }
  const preview = document.getElementById('decoded-preview');
  preview.src = b64;
  preview.style.display = 'block';
});
```

- [ ] **Step 2: Verify**

Open `http://localhost:3456/tools/image-to-base64.html`
- Upload a small PNG → Base64 string appears in textarea → Copy works
- Copy that Base64, paste in decode field → Preview Image → image appears below

- [ ] **Step 3: Commit**

```bash
git add tools/image-to-base64.html
git commit -m "feat: add Image to Base64 encoder/decoder"
```

---

## Task 16: Grayscale Converter

**Files:**
- Create: `tools/grayscale-converter.html`

- [ ] **Step 1: Create `tools/grayscale-converter.html`**

Tool header: icon `⚫`, title `Grayscale Converter`, desc `Convert any image to black and white. Adjust intensity with a slider.`

Include shared CSS and utils from Task 12.

**Tool interface HTML:**
```html
<div class="panel">
  <div class="drop-zone" id="drop-zone">
    <div class="drop-icon">📁</div>
    <p>Drop image here or click to upload</p>
    <input type="file" id="file-input" accept="image/*" style="display:none">
  </div>
</div>
<div id="tool-controls">
  <div class="panel">
    <div class="panel-label">Grayscale Intensity</div>
    <div style="display:flex;align-items:center;gap:14px">
      <input type="range" id="intensity-slider" min="0" max="100" value="100" style="flex:1;accent-color:var(--primary)">
      <span id="intensity-display" style="font-weight:700;color:var(--primary-light);min-width:42px">100%</span>
    </div>
    <p style="font-size:0.78rem;color:var(--muted);margin-top:6px">0% = original color · 100% = full grayscale</p>
  </div>
  <div class="panel">
    <div class="panel-label">Preview</div>
    <canvas id="preview-canvas"></canvas>
    <div class="btn-row" style="justify-content:center;margin-top:16px">
      <button class="btn btn-primary" id="download-btn">Download Grayscale Image</button>
    </div>
  </div>
</div>
```

**Related tools:** Image Resizer, Format Converter, Image Compressor

**JS logic:**
```js
let currentImg = null;
const canvas = document.getElementById('preview-canvas');
const ctx = canvas.getContext('2d');
const slider = document.getElementById('intensity-slider');

setupDropZone(document.getElementById('drop-zone'), document.getElementById('file-input'), (img) => {
  currentImg = img; applyGrayscale();
  document.getElementById('tool-controls').style.display = 'block';
});

slider.addEventListener('input', () => {
  document.getElementById('intensity-display').textContent = slider.value + '%';
  applyGrayscale();
});

function applyGrayscale() {
  if (!currentImg) return;
  canvas.width = currentImg.naturalWidth; canvas.height = currentImg.naturalHeight;
  ctx.drawImage(currentImg, 0, 0);
  const intensity = parseInt(slider.value) / 100;
  if (intensity === 0) return;
  const id = ctx.getImageData(0, 0, canvas.width, canvas.height);
  const d = id.data;
  for (let i = 0; i < d.length; i += 4) {
    const gray = 0.299*d[i] + 0.587*d[i+1] + 0.114*d[i+2];
    d[i]   = d[i]   + (gray - d[i])   * intensity;
    d[i+1] = d[i+1] + (gray - d[i+1]) * intensity;
    d[i+2] = d[i+2] + (gray - d[i+2]) * intensity;
  }
  ctx.putImageData(id, 0, 0);
}

document.getElementById('download-btn').addEventListener('click', () => { if(currentImg) dlCanvas(canvas,'grayscale.png'); });
```

- [ ] **Step 2: Verify**

Open `http://localhost:3456/tools/grayscale-converter.html`
- Upload a color JPG → canvas shows fully grayscale version (slider at 100%)
- Move slider to 50% → canvas shows partially desaturated image
- Download → `grayscale.png` downloads

- [ ] **Step 3: Commit**

```bash
git add tools/grayscale-converter.html
git commit -m "feat: add Grayscale Converter with intensity slider"
```

---

## Task 17: Image Cropper

**Files:**
- Create: `tools/image-cropper.html`

- [ ] **Step 1: Create `tools/image-cropper.html`**

Tool header: icon `✂️`, title `Image Cropper`, desc `Drag to select a crop area on the image preview, then download the cropped result.`

Include shared CSS and utils from Task 12. Add extra CSS:
```css
#preview-canvas { cursor: crosshair; }
.crop-info { font-size: 0.78rem; color: var(--muted); text-align: center; margin-top: 6px; }
```

**Tool interface HTML:**
```html
<div class="panel">
  <div class="drop-zone" id="drop-zone">
    <div class="drop-icon">📁</div>
    <p>Drop image here or click to upload</p>
    <small>Drag on the preview to select crop area</small>
    <input type="file" id="file-input" accept="image/*" style="display:none">
  </div>
</div>
<div id="tool-controls">
  <div class="panel">
    <div class="panel-label">Drag to select crop area</div>
    <canvas id="preview-canvas"></canvas>
    <div class="crop-info" id="crop-info">No crop selected yet. Drag on the image above.</div>
    <div class="btn-row" style="justify-content:center;margin-top:16px">
      <button class="btn btn-primary" id="crop-btn">Crop &amp; Download</button>
      <button class="btn btn-secondary" id="reset-btn">Reset Selection</button>
    </div>
  </div>
</div>
```

**Related tools:** Image Resizer, Format Converter, Grayscale Converter

**JS logic:**
```js
let currentImg = null, isDragging = false, startX = 0, startY = 0;
let cropX = 0, cropY = 0, cropW = 0, cropH = 0, scale = 1;
const canvas = document.getElementById('preview-canvas');
const ctx = canvas.getContext('2d');

setupDropZone(document.getElementById('drop-zone'), document.getElementById('file-input'), (img) => {
  currentImg = img;
  scale = Math.min(1, 700 / img.naturalWidth);
  canvas.width  = Math.round(img.naturalWidth  * scale);
  canvas.height = Math.round(img.naturalHeight * scale);
  ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
  cropW = 0; cropH = 0;
  document.getElementById('tool-controls').style.display = 'block';
});

function getPos(e) {
  const r = canvas.getBoundingClientRect();
  return { x: e.clientX - r.left, y: e.clientY - r.top };
}
function drawSelection() {
  ctx.drawImage(currentImg, 0, 0, canvas.width, canvas.height);
  if (cropW && cropH) {
    ctx.strokeStyle = '#6366f1'; ctx.lineWidth = 2; ctx.setLineDash([6,3]);
    ctx.strokeRect(cropX, cropY, cropW, cropH);
    ctx.fillStyle = 'rgba(99,102,241,0.12)';
    ctx.fillRect(cropX, cropY, cropW, cropH);
    const aw = Math.round(cropW/scale), ah = Math.round(cropH/scale);
    document.getElementById('crop-info').textContent = `Selected: ${aw}×${ah}px`;
  }
}
canvas.addEventListener('mousedown', e => { const p=getPos(e); startX=p.x; startY=p.y; isDragging=true; });
canvas.addEventListener('mousemove', e => {
  if (!isDragging) return;
  const p = getPos(e);
  cropX = Math.min(startX, p.x); cropY = Math.min(startY, p.y);
  cropW = Math.abs(p.x - startX); cropH = Math.abs(p.y - startY);
  drawSelection();
});
canvas.addEventListener('mouseup', () => { isDragging = false; });

document.getElementById('crop-btn').addEventListener('click', () => {
  if (!currentImg || !cropW || !cropH) { alert('Drag to select a crop area first.'); return; }
  const off = document.createElement('canvas');
  off.width  = Math.round(cropW / scale);
  off.height = Math.round(cropH / scale);
  off.getContext('2d').drawImage(currentImg, cropX/scale, cropY/scale, off.width, off.height, 0, 0, off.width, off.height);
  dlCanvas(off, 'cropped.png');
});
document.getElementById('reset-btn').addEventListener('click', () => {
  cropW = 0; cropH = 0;
  if (currentImg) ctx.drawImage(currentImg, 0, 0, canvas.width, canvas.height);
  document.getElementById('crop-info').textContent = 'No crop selected yet. Drag on the image above.';
});
```

- [ ] **Step 2: Verify**

Open `http://localhost:3456/tools/image-cropper.html`
- Upload an image → canvas shows scaled preview with crosshair cursor
- Drag a rectangle on the image → indigo dashed border shows selection, info shows dimensions
- Click "Crop & Download" → `cropped.png` downloads containing only the selected area
- Reset clears the selection

- [ ] **Step 3: Commit**

```bash
git add tools/image-cropper.html
git commit -m "feat: add Image Cropper with drag-to-select and canvas crop"
```

---

## Task 18: Final Polish — Reduced Motion + Verify All Links

**Files:**
- Modify: `index.html` (verify all 16 links point to correct files)
- All 16 tool pages (confirm related tool links are correct)

- [ ] **Step 1: Verify all 16 tool links on homepage**

Open `http://localhost:3456/index.html` and click each of the 16 cards. Each should navigate to the correct page and load without errors.

Expected mappings:
```
Word Counter        → tools/word-counter.html
Case Converter      → tools/case-converter.html
Lorem Ipsum         → tools/lorem-ipsum.html
Text Diff           → tools/text-diff.html
Remove Duplicates   → tools/remove-duplicates.html
Text Reverser       → tools/text-reverser.html
Whitespace Cleaner  → tools/whitespace-cleaner.html
Text to Slug        → tools/text-to-slug.html
Line Sorter         → tools/line-sorter.html
Find & Replace      → tools/find-replace.html
Image Resizer       → tools/image-resizer.html
Image Compressor    → tools/image-compressor.html
Format Converter    → tools/format-converter.html
Image to Base64     → tools/image-to-base64.html
Grayscale Converter → tools/grayscale-converter.html
Image Cropper       → tools/image-cropper.html
```

- [ ] **Step 2: Verify the `pointer-events: none` fix is present on all tool cards**

In `index.html`, confirm `.tool-card::before` and `.tool-card::after` both have `pointer-events: none` in the CSS. This prevents the shimmer/glow overlay from blocking clicks on card links.

Check in DevTools: right-click any card → Inspect → verify `pointer-events: none` on `::before` and `::after`.

- [ ] **Step 3: Add `prefers-reduced-motion` safeguard to `index.html` if not present**

At the end of the `<style>` block in `index.html`, confirm this rule exists:
```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after { animation-duration: 0.01ms !important; transition-duration: 0.01ms !important; }
}
```

- [ ] **Step 4: Commit**

```bash
git add index.html tools/
git commit -m "chore: final polish — verify all links, pointer-events fix, reduced-motion"
```

---

## Task 19: Deploy to Vercel

- [ ] **Step 1: Push all commits to GitHub**

```bash
git push origin master
```

- [ ] **Step 2: Deploy to Vercel**

The project is already linked to Vercel from the Lumina AI deploy. Run:
```bash
vercel --prod
```

If prompted for project name, use `aiwowtools`.

- [ ] **Step 3: Verify live URL**

Open the Vercel URL. Test:
- Homepage loads with all 16 tool cards
- Click "Word Counter" → navigates to tool page
- Drag a file onto Image Resizer → upload works
- Search "slug" → only Text to Slug card shows

- [ ] **Step 4: Commit deployment confirmation**

```bash
git commit --allow-empty -m "chore: deployed AIWowTools to Vercel"
```
