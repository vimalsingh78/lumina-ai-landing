# Lumina AI Landing Page — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a single `index.html` premium animated landing page for Lumina AI, an AI & Data learning platform, with GSAP scroll animations, custom cursor, and cinematic design.

**Architecture:** One self-contained `index.html` with embedded `<style>` and `<script>` blocks. All libraries loaded via CDN (GSAP, Lenis, Google Fonts). No build step required — open directly in a browser.

**Tech Stack:** HTML5, CSS3 (custom properties, animations, glassmorphism), Vanilla JS (ES6+), GSAP 3 + ScrollTrigger, Lenis smooth scroll, Google Fonts (Space Grotesk + Inter), HTML5 Canvas

---

## File Map

| File | Role |
|------|------|
| `index.html` | Entire page: markup + embedded CSS + embedded JS |

---

### Task 1: Base HTML Scaffold + CSS Foundation

**Files:**
- Create: `index.html`

- [ ] **Step 1: Create the base HTML file with CDN links and CSS variables**

Create `index.html` with this exact content:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Lumina AI — Master AI & Data Science</title>
  <link rel="preconnect" href="https://fonts.googleapis.com" />
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
  <link href="https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@400;500;600;700;800&family=Inter:wght@400;500;600&display=swap" rel="stylesheet" />
  <style>
    /* ── CSS CUSTOM PROPERTIES ── */
    :root {
      --bg: #020617;
      --surface: #0f172a;
      --border: #1e293b;
      --primary: #6366f1;
      --primary-light: #818cf8;
      --accent: #f59e0b;
      --text: #f1f5f9;
      --muted: #64748b;
      --font-display: 'Space Grotesk', sans-serif;
      --font-body: 'Inter', sans-serif;
    }

    /* ── RESET ── */
    *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
    html { scroll-behavior: auto; } /* Lenis handles scrolling */
    body {
      background: var(--bg);
      color: var(--text);
      font-family: var(--font-body);
      overflow-x: hidden;
      cursor: none; /* custom cursor replaces default */
    }
    a { color: inherit; text-decoration: none; }
    img { max-width: 100%; display: block; }

    /* ── UTILITY ── */
    .container {
      max-width: 1200px;
      margin: 0 auto;
      padding: 0 24px;
    }
    .gradient-text {
      background: linear-gradient(135deg, var(--primary-light), var(--accent));
      -webkit-background-clip: text;
      -webkit-text-fill-color: transparent;
      background-clip: text;
    }
    .section-label {
      display: inline-block;
      font-size: 0.75rem;
      font-weight: 700;
      letter-spacing: 0.15em;
      text-transform: uppercase;
      color: var(--primary-light);
      margin-bottom: 16px;
    }
    .section-title {
      font-family: var(--font-display);
      font-size: clamp(2rem, 4vw, 3rem);
      font-weight: 700;
      line-height: 1.15;
      margin-bottom: 16px;
    }
    .section-subtitle {
      font-size: 1.125rem;
      color: var(--muted);
      max-width: 560px;
      line-height: 1.7;
    }
  </style>
</head>
<body>
  <!-- sections go here -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.5/gsap.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.5/ScrollTrigger.min.js"></script>
  <script src="https://unpkg.com/lenis@1.1.14/dist/lenis.min.js"></script>
  <script>
    gsap.registerPlugin(ScrollTrigger);
    // JS goes here
  </script>
</body>
</html>
```

- [ ] **Step 2: Open in browser and verify**

Open `index.html` in a browser. Expected: blank dark (`#020617`) page, no console errors, fonts loading (check Network tab).

---

### Task 2: Custom Cursor

**Files:**
- Modify: `index.html` — add cursor HTML, CSS, and JS

- [ ] **Step 1: Add cursor markup after `<body>` opens**

Insert immediately after `<body>`:
```html
<!-- Custom Cursor -->
<div id="cursor-dot"></div>
<div id="cursor-ring"></div>
```

- [ ] **Step 2: Add cursor CSS inside the `<style>` block**

Append inside `<style>`:
```css
/* ── CUSTOM CURSOR ── */
#cursor-dot, #cursor-ring {
  position: fixed;
  top: 0; left: 0;
  pointer-events: none;
  z-index: 9999;
  border-radius: 50%;
  transform: translate(-50%, -50%);
  transition: opacity 0.2s;
}
#cursor-dot {
  width: 8px; height: 8px;
  background: var(--primary);
}
#cursor-ring {
  width: 32px; height: 32px;
  border: 1.5px solid rgba(99,102,241,0.5);
  transition: width 0.2s, height 0.2s, background 0.2s;
}
body.cursor-hover #cursor-dot { opacity: 0; }
body.cursor-hover #cursor-ring {
  width: 48px; height: 48px;
  background: rgba(99,102,241,0.1);
}
@media (hover: none) {
  #cursor-dot, #cursor-ring { display: none; }
  body { cursor: auto; }
}
```

- [ ] **Step 3: Add cursor JS inside `<script>` block (after gsap.registerPlugin)**

```javascript
// ── CUSTOM CURSOR ──
const dot = document.getElementById('cursor-dot');
const ring = document.getElementById('cursor-ring');
let mouseX = 0, mouseY = 0;
let ringX = 0, ringY = 0;

document.addEventListener('mousemove', e => {
  mouseX = e.clientX;
  mouseY = e.clientY;
  gsap.set(dot, { x: mouseX, y: mouseY });
});

// Ring follows with lerp
gsap.ticker.add(() => {
  ringX += (mouseX - ringX) * 0.12;
  ringY += (mouseY - ringY) * 0.12;
  gsap.set(ring, { x: ringX, y: ringY });
});

document.querySelectorAll('a, button, [data-cursor-hover]').forEach(el => {
  el.addEventListener('mouseenter', () => document.body.classList.add('cursor-hover'));
  el.addEventListener('mouseleave', () => document.body.classList.remove('cursor-hover'));
});
```

- [ ] **Step 4: Verify in browser**

Move mouse around the page. Expected: small indigo dot tracks exactly, larger semi-transparent ring follows with slight lag.

---

### Task 3: Navigation

**Files:**
- Modify: `index.html` — add nav HTML, CSS, JS

- [ ] **Step 1: Add nav markup (replace `<!-- sections go here -->` comment)**

```html
<!-- Navigation -->
<nav id="navbar">
  <div class="container nav-inner">
    <a href="#" class="nav-logo">
      <span class="nav-logo-text">Lumina</span><span class="nav-logo-dot">AI</span>
    </a>
    <ul class="nav-links">
      <li><a href="#courses">Courses</a></li>
      <li><a href="#features">Features</a></li>
      <li><a href="#testimonials">Testimonials</a></li>
      <li><a href="#cta">Pricing</a></li>
    </ul>
    <a href="#cta" class="btn btn-primary">Start Learning</a>
  </div>
</nav>
```

- [ ] **Step 2: Add nav CSS**

Append inside `<style>`:
```css
/* ── NAVIGATION ── */
#navbar {
  position: fixed;
  top: 0; left: 0; right: 0;
  z-index: 100;
  padding: 20px 0;
  transition: background 0.4s, backdrop-filter 0.4s, border-color 0.4s;
  border-bottom: 1px solid transparent;
}
#navbar.scrolled {
  background: rgba(2, 6, 23, 0.8);
  backdrop-filter: blur(16px);
  -webkit-backdrop-filter: blur(16px);
  border-bottom-color: var(--border);
  padding: 14px 0;
}
.nav-inner {
  display: flex;
  align-items: center;
  justify-content: space-between;
}
.nav-logo { display: flex; align-items: center; gap: 2px; }
.nav-logo-text {
  font-family: var(--font-display);
  font-size: 1.25rem;
  font-weight: 800;
  color: var(--text);
}
.nav-logo-dot {
  font-family: var(--font-display);
  font-size: 1.25rem;
  font-weight: 800;
  color: var(--primary);
  margin-left: 2px;
}
.nav-links {
  display: flex;
  gap: 36px;
  list-style: none;
}
.nav-links a {
  font-size: 0.9rem;
  font-weight: 500;
  color: var(--muted);
  transition: color 0.2s;
}
.nav-links a:hover { color: var(--text); }

/* ── BUTTONS ── */
.btn {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 10px 22px;
  border-radius: 100px;
  font-size: 0.875rem;
  font-weight: 600;
  font-family: var(--font-body);
  border: none;
  cursor: none;
  transition: transform 0.2s, box-shadow 0.2s;
}
.btn-primary {
  background: linear-gradient(135deg, var(--primary), #4f46e5);
  color: #fff;
  box-shadow: 0 0 20px rgba(99,102,241,0.3);
}
.btn-primary:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 32px rgba(99,102,241,0.5);
}
.btn-outline {
  background: transparent;
  color: var(--text);
  border: 1px solid var(--border);
}
.btn-outline:hover {
  border-color: var(--primary);
  color: var(--primary-light);
}
@media (max-width: 768px) {
  .nav-links { display: none; }
}
```

- [ ] **Step 3: Add nav scroll JS (append inside `<script>` block)**

```javascript
// ── NAV SCROLL STATE ──
const navbar = document.getElementById('navbar');
window.addEventListener('scroll', () => {
  navbar.classList.toggle('scrolled', window.scrollY > 40);
}, { passive: true });

// Fade-in on load
gsap.from('#navbar', { y: -20, opacity: 0, duration: 0.8, ease: 'power2.out', delay: 0.2 });
```

- [ ] **Step 4: Verify in browser**

Expected: nav appears at top, transparent. Scrolling down (once hero is built) will trigger glassmorphism. "Start Learning" button renders with indigo gradient.

---

### Task 4: Hero Section — Markup & CSS

**Files:**
- Modify: `index.html` — add hero HTML and CSS

- [ ] **Step 1: Add hero markup (after the nav element)**

```html
<!-- Hero -->
<section id="hero">
  <canvas id="particle-canvas"></canvas>
  <div class="hero-glow"></div>
  <div class="container hero-content">
    <div id="hero-badge">✦ 50,000+ Students Enrolled</div>
    <h1 id="hero-headline">
      <span class="hero-line">Master AI &amp;</span>
      <span class="hero-line gradient-text">Data Science</span>
    </h1>
    <p id="hero-sub">Learn cutting-edge AI, machine learning, and data engineering from industry experts. Build real projects. Get certified.</p>
    <div class="hero-cta-row">
      <button id="hero-btn" class="btn btn-primary btn-lg" data-cursor-hover>
        Start Learning Free →
      </button>
    </div>
    <p class="hero-trust">No credit card · 14-day free trial · Cancel anytime</p>
  </div>
  <div class="hero-scroll-indicator">
    <div class="scroll-chevron"></div>
  </div>
</section>
```

- [ ] **Step 2: Add hero CSS (append inside `<style>`)**

```css
/* ── HERO ── */
#hero {
  position: relative;
  min-height: 100vh;
  display: flex;
  align-items: center;
  overflow: hidden;
}
#particle-canvas {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
}
.hero-glow {
  position: absolute;
  top: 30%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 600px;
  height: 400px;
  background: radial-gradient(ellipse, rgba(99,102,241,0.18) 0%, transparent 70%);
  pointer-events: none;
}
.hero-content {
  position: relative;
  z-index: 2;
  text-align: center;
  padding-top: 100px;
}
#hero-badge {
  display: inline-block;
  border: 1px solid rgba(99,102,241,0.4);
  border-radius: 100px;
  padding: 6px 16px;
  font-size: 0.8rem;
  font-weight: 600;
  color: var(--primary-light);
  letter-spacing: 0.05em;
  margin-bottom: 32px;
  background: rgba(99,102,241,0.08);
  opacity: 0;
}
#hero-headline {
  font-family: var(--font-display);
  font-size: clamp(3rem, 8vw, 6rem);
  font-weight: 800;
  line-height: 1.05;
  margin-bottom: 24px;
}
.hero-line {
  display: block;
  opacity: 0;
  transform: translateY(40px);
}
#hero-sub {
  font-size: clamp(1rem, 2vw, 1.25rem);
  color: var(--muted);
  max-width: 560px;
  margin: 0 auto 36px;
  line-height: 1.7;
  opacity: 0;
}
.hero-cta-row { margin-bottom: 16px; opacity: 0; }
.btn-lg { padding: 14px 32px; font-size: 1rem; }
.hero-trust {
  font-size: 0.8rem;
  color: var(--muted);
  opacity: 0;
}
.hero-scroll-indicator {
  position: absolute;
  bottom: 40px;
  left: 50%;
  transform: translateX(-50%);
  opacity: 0;
}
.scroll-chevron {
  width: 20px;
  height: 20px;
  border-right: 2px solid var(--muted);
  border-bottom: 2px solid var(--muted);
  transform: rotate(45deg);
  animation: chevron-bounce 1.5s ease-in-out infinite;
}
@keyframes chevron-bounce {
  0%, 100% { transform: rotate(45deg) translateY(0); }
  50% { transform: rotate(45deg) translateY(6px); }
}
```

- [ ] **Step 3: Verify layout in browser**

Expected: dark full-screen hero, centered content area visible (all invisible due to `opacity: 0` — animations come next), chevron at bottom.

---

### Task 5: Canvas Particle System

**Files:**
- Modify: `index.html` — add particle JS

- [ ] **Step 1: Add particle system JS (append inside `<script>` block)**

```javascript
// ── CANVAS PARTICLES ──
(function() {
  const canvas = document.getElementById('particle-canvas');
  const ctx = canvas.getContext('2d');
  let particles = [];
  const COUNT = 80;
  const MAX_DIST = 120;

  function resize() {
    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;
  }
  resize();
  window.addEventListener('resize', resize, { passive: true });

  for (let i = 0; i < COUNT; i++) {
    particles.push({
      x: Math.random() * canvas.width,
      y: Math.random() * canvas.height,
      vx: (Math.random() - 0.5) * 0.4,
      vy: (Math.random() - 0.5) * 0.4,
      r: Math.random() * 2 + 1,
    });
  }

  function draw() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    particles.forEach(p => {
      p.x += p.vx;
      p.y += p.vy;
      if (p.x < 0 || p.x > canvas.width) p.vx *= -1;
      if (p.y < 0 || p.y > canvas.height) p.vy *= -1;

      ctx.beginPath();
      ctx.arc(p.x, p.y, p.r, 0, Math.PI * 2);
      ctx.fillStyle = 'rgba(99,102,241,0.6)';
      ctx.fill();
    });

    for (let i = 0; i < particles.length; i++) {
      for (let j = i + 1; j < particles.length; j++) {
        const dx = particles[i].x - particles[j].x;
        const dy = particles[i].y - particles[j].y;
        const dist = Math.sqrt(dx * dx + dy * dy);
        if (dist < MAX_DIST) {
          ctx.beginPath();
          ctx.moveTo(particles[i].x, particles[i].y);
          ctx.lineTo(particles[j].x, particles[j].y);
          ctx.strokeStyle = `rgba(99,102,241,${0.15 * (1 - dist / MAX_DIST)})`;
          ctx.lineWidth = 0.5;
          ctx.stroke();
        }
      }
    }
    requestAnimationFrame(draw);
  }
  draw();
})();
```

- [ ] **Step 2: Verify in browser**

Expected: animated network of floating indigo dots with connecting lines on the hero background.

---

### Task 6: Hero Animations (GSAP)

**Files:**
- Modify: `index.html` — add hero entrance animations and parallax

- [ ] **Step 1: Add hero GSAP animations (append inside `<script>` block)**

```javascript
// ── HERO ANIMATIONS ──
const heroTl = gsap.timeline({ delay: 0.3 });
heroTl
  .to('#hero-badge',     { opacity: 1, y: 0, duration: 0.6, ease: 'power2.out' })
  .to('.hero-line',      { opacity: 1, y: 0, duration: 0.7, ease: 'power3.out', stagger: 0.15 }, '-=0.2')
  .to('#hero-sub',       { opacity: 1, duration: 0.6, ease: 'power2.out' }, '-=0.3')
  .to('.hero-cta-row',   { opacity: 1, y: 0, duration: 0.5, ease: 'power2.out' }, '-=0.2')
  .to('.hero-trust',     { opacity: 1, duration: 0.4, ease: 'power2.out' }, '-=0.1')
  .to('.hero-scroll-indicator', { opacity: 1, duration: 0.4 }, '-=0.1');

// Parallax: hero content moves up slightly on scroll
gsap.to('.hero-content', {
  yPercent: -20,
  ease: 'none',
  scrollTrigger: {
    trigger: '#hero',
    start: 'top top',
    end: 'bottom top',
    scrub: true,
  }
});
```

- [ ] **Step 2: Verify in browser**

Expected: on page load, badge fades in, then headline lines slide up one by one, then subtext and CTA appear. Scrolling down makes hero content drift upward (parallax).

---

### Task 7: Magnetic Button Effect

**Files:**
- Modify: `index.html` — add magnetic hover JS

- [ ] **Step 1: Add magnetic button JS (append inside `<script>` block)**

```javascript
// ── MAGNETIC BUTTON ──
(function() {
  const btn = document.getElementById('hero-btn');
  if (!btn) return;
  btn.addEventListener('mousemove', e => {
    const rect = btn.getBoundingClientRect();
    const bx = rect.left + rect.width / 2;
    const by = rect.top + rect.height / 2;
    const dx = e.clientX - bx;
    const dy = e.clientY - by;
    gsap.to(btn, { x: dx * 0.3, y: dy * 0.3, duration: 0.3, ease: 'power2.out' });
  });
  btn.addEventListener('mouseleave', () => {
    gsap.to(btn, { x: 0, y: 0, duration: 0.5, ease: 'elastic.out(1, 0.4)' });
  });
})();
```

- [ ] **Step 2: Verify in browser**

Expected: hovering near the "Start Learning Free" button causes it to pull slightly toward the cursor; on mouseleave it snaps back with an elastic bounce.

---

### Task 8: Stats Bar

**Files:**
- Modify: `index.html` — add stats section HTML, CSS, and JS

- [ ] **Step 1: Add stats markup (after closing `</section>` of hero)**

```html
<!-- Stats Bar -->
<section id="stats">
  <div class="container">
    <div class="stats-grid">
      <div class="stat-item" data-count="50000" data-suffix="+" data-label="Students Enrolled">
        <div class="stat-number"><span class="stat-val">0</span><span class="stat-sfx">+</span></div>
        <div class="stat-label">Students Enrolled</div>
      </div>
      <div class="stat-item" data-count="200" data-suffix="+" data-label="Expert Courses">
        <div class="stat-number"><span class="stat-val">0</span><span class="stat-sfx">+</span></div>
        <div class="stat-label">Expert Courses</div>
      </div>
      <div class="stat-item" data-count="95" data-suffix="%" data-label="Completion Rate">
        <div class="stat-number"><span class="stat-val">0</span><span class="stat-sfx">%</span></div>
        <div class="stat-label">Completion Rate</div>
      </div>
      <div class="stat-item" data-count="4.9" data-suffix="★" data-label="Average Rating" data-decimals="1">
        <div class="stat-number"><span class="stat-val">0</span><span class="stat-sfx">★</span></div>
        <div class="stat-label">Average Rating</div>
      </div>
    </div>
  </div>
</section>
```

- [ ] **Step 2: Add stats CSS**

```css
/* ── STATS BAR ── */
#stats {
  padding: 64px 0;
  border-top: 1px solid var(--border);
  border-bottom: 1px solid var(--border);
  background: linear-gradient(180deg, rgba(99,102,241,0.04) 0%, transparent 100%);
}
.stats-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 0;
}
.stat-item {
  text-align: center;
  padding: 24px;
  border-right: 1px solid var(--border);
}
.stat-item:last-child { border-right: none; }
.stat-number {
  font-family: var(--font-display);
  font-size: clamp(2rem, 4vw, 3rem);
  font-weight: 800;
  color: var(--text);
  display: flex;
  align-items: baseline;
  justify-content: center;
  gap: 2px;
}
.stat-sfx { color: var(--primary-light); font-size: 0.6em; }
.stat-label {
  font-size: 0.85rem;
  color: var(--muted);
  margin-top: 8px;
  font-weight: 500;
}
@media (max-width: 640px) {
  .stats-grid { grid-template-columns: repeat(2, 1fr); }
  .stat-item:nth-child(2) { border-right: none; }
  .stat-item:nth-child(3) { border-top: 1px solid var(--border); }
  .stat-item:nth-child(4) { border-top: 1px solid var(--border); border-right: none; }
}
```

- [ ] **Step 3: Add counter animation JS**

```javascript
// ── STATS COUNTER ──
document.querySelectorAll('.stat-item').forEach(item => {
  const valEl = item.querySelector('.stat-val');
  const target = parseFloat(item.dataset.count);
  const decimals = parseInt(item.dataset.decimals || '0');
  const obj = { val: 0 };

  ScrollTrigger.create({
    trigger: item,
    start: 'top 85%',
    once: true,
    onEnter: () => {
      gsap.to(obj, {
        val: target,
        duration: 2,
        ease: 'power2.out',
        onUpdate: () => {
          valEl.textContent = decimals
            ? obj.val.toFixed(decimals)
            : Math.round(obj.val).toLocaleString();
        }
      });
    }
  });
});
```

- [ ] **Step 4: Verify in browser**

Expected: 4-column stats bar below hero. Scrolling to it triggers numbers counting up from 0. On mobile, 2×2 grid.

---

### Task 9: Feature Courses Section

**Files:**
- Modify: `index.html` — add courses section HTML, CSS, and scroll animations

- [ ] **Step 1: Add courses markup (after stats section)**

```html
<!-- Courses -->
<section id="courses">
  <div class="container">
    <div class="section-header">
      <span class="section-label">✦ Curriculum</span>
      <h2 class="section-title">What You'll <span class="gradient-text">Master</span></h2>
      <p class="section-subtitle">Hands-on courses designed by AI practitioners. Real projects, not just theory.</p>
    </div>
    <div class="courses-grid">
      <div class="course-card" data-cursor-hover>
        <div class="course-icon">🧠</div>
        <div class="course-badge">Beginner</div>
        <h3 class="course-title">Machine Learning Fundamentals</h3>
        <p class="course-desc">Build your first models. Supervised, unsupervised, and reinforcement learning from the ground up.</p>
        <a href="#" class="course-link">Explore Course →</a>
      </div>
      <div class="course-card" data-cursor-hover>
        <div class="course-icon">⚡</div>
        <div class="course-badge course-badge--advanced">Advanced</div>
        <h3 class="course-title">Deep Learning & Neural Networks</h3>
        <p class="course-desc">Master CNNs, RNNs, Transformers, and modern architectures used in production AI systems.</p>
        <a href="#" class="course-link">Explore Course →</a>
      </div>
      <div class="course-card" data-cursor-hover>
        <div class="course-icon">📊</div>
        <div class="course-badge">Beginner</div>
        <h3 class="course-title">Data Science with Python</h3>
        <p class="course-desc">Pandas, NumPy, Matplotlib, and Scikit-learn. Turn raw data into actionable insights.</p>
        <a href="#" class="course-link">Explore Course →</a>
      </div>
      <div class="course-card" data-cursor-hover>
        <div class="course-icon">💬</div>
        <div class="course-badge course-badge--intermediate">Intermediate</div>
        <h3 class="course-title">Natural Language Processing</h3>
        <p class="course-desc">Build LLM-powered apps, fine-tune language models, and deploy NLP pipelines at scale.</p>
        <a href="#" class="course-link">Explore Course →</a>
      </div>
      <div class="course-card" data-cursor-hover>
        <div class="course-icon">👁️</div>
        <div class="course-badge course-badge--intermediate">Intermediate</div>
        <h3 class="course-title">Computer Vision</h3>
        <p class="course-desc">Object detection, image segmentation, and real-time vision systems with PyTorch and OpenCV.</p>
        <a href="#" class="course-link">Explore Course →</a>
      </div>
      <div class="course-card" data-cursor-hover>
        <div class="course-icon">🔧</div>
        <div class="course-badge course-badge--advanced">Advanced</div>
        <h3 class="course-title">Data Engineering & MLOps</h3>
        <p class="course-desc">Build production pipelines, CI/CD for ML, Airflow, dbt, and containerized model serving.</p>
        <a href="#" class="course-link">Explore Course →</a>
      </div>
    </div>
  </div>
</section>
```

- [ ] **Step 2: Add courses CSS**

```css
/* ── COURSES ── */
#courses {
  padding: 120px 0;
}
.section-header {
  text-align: center;
  margin-bottom: 72px;
}
.section-header .section-subtitle {
  margin: 0 auto;
}
.courses-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 24px;
}
.course-card {
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: 16px;
  padding: 32px;
  position: relative;
  overflow: hidden;
  transition: transform 0.3s, border-color 0.3s, box-shadow 0.3s;
  opacity: 0;
  transform: scale(0.85) translateY(20px);
}
.course-card::before {
  content: '';
  position: absolute;
  inset: 0;
  background: linear-gradient(135deg, rgba(99,102,241,0.06) 0%, transparent 60%);
  opacity: 0;
  transition: opacity 0.3s;
}
.course-card:hover {
  transform: translateY(-8px) scale(1.01);
  border-color: var(--primary);
  box-shadow: 0 0 40px rgba(99,102,241,0.2);
}
.course-card:hover::before { opacity: 1; }
.course-card::after {
  content: '';
  position: absolute;
  top: 0; left: -100%;
  width: 60%;
  height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255,255,255,0.04), transparent);
  transition: left 0.5s ease;
}
.course-card:hover::after { left: 150%; }
.course-icon {
  font-size: 2rem;
  margin-bottom: 16px;
}
.course-badge {
  display: inline-block;
  font-size: 0.7rem;
  font-weight: 700;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  padding: 3px 10px;
  border-radius: 100px;
  background: rgba(99,102,241,0.15);
  color: var(--primary-light);
  margin-bottom: 12px;
}
.course-badge--intermediate {
  background: rgba(245,158,11,0.12);
  color: #fcd34d;
}
.course-badge--advanced {
  background: rgba(239,68,68,0.12);
  color: #fca5a5;
}
.course-title {
  font-family: var(--font-display);
  font-size: 1.1rem;
  font-weight: 700;
  margin-bottom: 10px;
  line-height: 1.3;
}
.course-desc {
  font-size: 0.875rem;
  color: var(--muted);
  line-height: 1.7;
  margin-bottom: 20px;
}
.course-link {
  font-size: 0.85rem;
  font-weight: 600;
  color: var(--primary-light);
  transition: color 0.2s, gap 0.2s;
}
.course-link:hover { color: var(--accent); }
@media (max-width: 900px) {
  .courses-grid { grid-template-columns: repeat(2, 1fr); }
}
@media (max-width: 600px) {
  .courses-grid { grid-template-columns: 1fr; }
}
```

- [ ] **Step 3: Add staggered scroll reveal for cards**

```javascript
// ── COURSE CARDS REVEAL ──
gsap.to('.course-card', {
  opacity: 1,
  scale: 1,
  y: 0,
  duration: 0.7,
  ease: 'power3.out',
  stagger: 0.1,
  scrollTrigger: {
    trigger: '.courses-grid',
    start: 'top 80%',
  }
});
```

- [ ] **Step 4: Verify in browser**

Expected: scroll to courses section → 6 cards reveal one by one, fading in and scaling up. Hover on a card lifts it with indigo glow border and shimmer sweep.

---

### Task 10: How It Works Section

**Files:**
- Modify: `index.html` — add timeline section

- [ ] **Step 1: Add how-it-works markup (after courses section)**

```html
<!-- How It Works -->
<section id="features">
  <div class="container">
    <div class="section-header">
      <span class="section-label">✦ Process</span>
      <h2 class="section-title">Your Learning <span class="gradient-text">Journey</span></h2>
    </div>
    <div class="timeline">
      <div class="timeline-line"><div class="timeline-line-fill"></div></div>
      <div class="timeline-steps">
        <div class="timeline-step">
          <div class="step-number">01</div>
          <div class="step-icon">🎯</div>
          <h3 class="step-title">Choose Your Path</h3>
          <p class="step-desc">Take a 2-minute assessment. We map your goals to a personalized learning path across AI, ML, and Data tracks.</p>
        </div>
        <div class="timeline-step">
          <div class="step-number">02</div>
          <div class="step-icon">💻</div>
          <h3 class="step-title">Learn by Doing</h3>
          <p class="step-desc">Interactive notebooks, real datasets, and AI-assisted feedback. No passive video watching — pure hands-on practice.</p>
        </div>
        <div class="timeline-step">
          <div class="step-number">03</div>
          <div class="step-icon">🏆</div>
          <h3 class="step-title">Get Certified</h3>
          <p class="step-desc">Earn industry-recognized certificates. Showcase your portfolio on LinkedIn. Get hired faster.</p>
        </div>
      </div>
    </div>
  </div>
</section>
```

- [ ] **Step 2: Add timeline CSS**

```css
/* ── HOW IT WORKS ── */
#features {
  padding: 120px 0;
  background: linear-gradient(180deg, transparent 0%, rgba(99,102,241,0.04) 50%, transparent 100%);
}
.timeline {
  position: relative;
  margin-top: 64px;
}
.timeline-line {
  position: absolute;
  top: 60px;
  left: 16.67%;
  right: 16.67%;
  height: 2px;
  background: var(--border);
  overflow: hidden;
}
.timeline-line-fill {
  width: 0%;
  height: 100%;
  background: linear-gradient(90deg, var(--primary), var(--accent));
  transition: width 0.1s;
}
.timeline-steps {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 32px;
  position: relative;
}
.timeline-step {
  text-align: center;
  padding: 0 16px;
  opacity: 0;
  transform: translateY(30px);
}
.step-number {
  font-family: var(--font-display);
  font-size: 0.75rem;
  font-weight: 700;
  color: var(--primary);
  letter-spacing: 0.1em;
  margin-bottom: 12px;
}
.step-icon {
  font-size: 2.5rem;
  margin-bottom: 16px;
  display: inline-block;
}
.step-title {
  font-family: var(--font-display);
  font-size: 1.25rem;
  font-weight: 700;
  margin-bottom: 12px;
}
.step-desc {
  font-size: 0.9rem;
  color: var(--muted);
  line-height: 1.7;
}
@media (max-width: 768px) {
  .timeline-steps { grid-template-columns: 1fr; }
  .timeline-line { display: none; }
}
```

- [ ] **Step 3: Add timeline scroll animation JS**

```javascript
// ── TIMELINE ANIMATION ──
gsap.to('.timeline-step', {
  opacity: 1,
  y: 0,
  duration: 0.7,
  ease: 'power3.out',
  stagger: 0.2,
  scrollTrigger: {
    trigger: '.timeline-steps',
    start: 'top 80%',
  }
});

// Line draw animation
ScrollTrigger.create({
  trigger: '.timeline',
  start: 'top 75%',
  end: 'bottom 60%',
  scrub: 1,
  onUpdate: self => {
    document.querySelector('.timeline-line-fill').style.width = (self.progress * 100) + '%';
  }
});
```

- [ ] **Step 4: Verify in browser**

Expected: 3-step timeline appears. Line draws from left to right as section scrolls through viewport. Steps fade up on enter.

---

### Task 11: Testimonials Section

**Files:**
- Modify: `index.html` — add testimonials

- [ ] **Step 1: Add testimonials markup (after how-it-works)**

```html
<!-- Testimonials -->
<section id="testimonials">
  <div class="container">
    <div class="section-header">
      <span class="section-label">✦ Reviews</span>
      <h2 class="section-title">Loved by <span class="gradient-text">Learners</span></h2>
      <p class="section-subtitle">Join thousands of professionals who transformed their careers with Lumina AI.</p>
    </div>
    <div class="testimonials-grid">
      <div class="testimonial-card" data-parallax-offset="-20">
        <div class="testimonial-stars">★★★★★</div>
        <p class="testimonial-quote">"The deep learning course completely changed my career. Within 3 months of completing the MLOps track, I landed a senior ML engineer role. Best investment I've made."</p>
        <div class="testimonial-author">
          <div class="author-avatar" style="background: linear-gradient(135deg, #6366f1, #8b5cf6);">SJ</div>
          <div>
            <div class="author-name">Sarah Johnson</div>
            <div class="author-role">ML Engineer @ Google</div>
          </div>
        </div>
      </div>
      <div class="testimonial-card" data-parallax-offset="0">
        <div class="testimonial-stars">★★★★★</div>
        <p class="testimonial-quote">"I went from knowing nothing about data science to building production pipelines. The hands-on projects are incredibly well designed. Lumina AI is in a class of its own."</p>
        <div class="testimonial-author">
          <div class="author-avatar" style="background: linear-gradient(135deg, #f59e0b, #ef4444);">MC</div>
          <div>
            <div class="author-name">Marcus Chen</div>
            <div class="author-role">Data Scientist @ Stripe</div>
          </div>
        </div>
      </div>
      <div class="testimonial-card" data-parallax-offset="20">
        <div class="testimonial-stars">★★★★★</div>
        <p class="testimonial-quote">"The NLP and Computer Vision tracks are absolutely world-class. I've taken Coursera, edX, Fast.ai — nothing compares. The AI-assisted feedback alone is worth the price."</p>
        <div class="testimonial-author">
          <div class="author-avatar" style="background: linear-gradient(135deg, #10b981, #06b6d4);">AP</div>
          <div>
            <div class="author-name">Aisha Patel</div>
            <div class="author-role">AI Researcher @ OpenAI</div>
          </div>
        </div>
      </div>
    </div>
  </div>
</section>
```

- [ ] **Step 2: Add testimonials CSS**

```css
/* ── TESTIMONIALS ── */
#testimonials {
  padding: 120px 0;
}
.testimonials-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 24px;
  align-items: start;
}
.testimonial-card {
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: 16px;
  padding: 32px;
  opacity: 0;
  transform: translateY(40px);
  will-change: transform;
}
.testimonial-stars {
  color: var(--accent);
  font-size: 0.9rem;
  letter-spacing: 2px;
  margin-bottom: 16px;
}
.testimonial-quote {
  font-size: 0.95rem;
  line-height: 1.75;
  color: #cbd5e1;
  margin-bottom: 24px;
  font-style: italic;
}
.testimonial-author {
  display: flex;
  align-items: center;
  gap: 12px;
}
.author-avatar {
  width: 44px; height: 44px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 0.8rem;
  font-weight: 700;
  color: white;
  flex-shrink: 0;
}
.author-name {
  font-weight: 600;
  font-size: 0.9rem;
}
.author-role {
  font-size: 0.8rem;
  color: var(--muted);
  margin-top: 2px;
}
@media (max-width: 900px) {
  .testimonials-grid { grid-template-columns: 1fr; }
}
```

- [ ] **Step 3: Add testimonials scroll animations with parallax**

```javascript
// ── TESTIMONIALS ──
gsap.to('.testimonial-card', {
  opacity: 1,
  y: 0,
  duration: 0.8,
  ease: 'power3.out',
  stagger: 0.15,
  scrollTrigger: {
    trigger: '.testimonials-grid',
    start: 'top 80%',
  }
});

// Subtle parallax depth
document.querySelectorAll('.testimonial-card').forEach(card => {
  const offset = parseFloat(card.dataset.parallaxOffset || 0);
  gsap.to(card, {
    y: offset,
    ease: 'none',
    scrollTrigger: {
      trigger: '#testimonials',
      start: 'top bottom',
      end: 'bottom top',
      scrub: 1.5,
    }
  });
});
```

- [ ] **Step 4: Verify in browser**

Expected: 3 testimonial cards animate in on scroll. As you continue scrolling, the three cards move at slightly different vertical speeds creating a parallax depth illusion.

---

### Task 12: CTA Banner Section

**Files:**
- Modify: `index.html` — add CTA section

- [ ] **Step 1: Add CTA markup (after testimonials)**

```html
<!-- CTA -->
<section id="cta">
  <div class="cta-bg-glow"></div>
  <div class="container cta-inner">
    <span class="section-label">✦ Get Started Today</span>
    <h2 class="cta-title">Start Your <span class="gradient-text">AI Journey</span> Today</h2>
    <p class="cta-subtitle">Join 50,000+ learners. Free 14-day trial. No credit card required.</p>
    <form class="cta-form" onsubmit="return false;">
      <input type="email" class="cta-input" placeholder="Enter your email address" />
      <button type="submit" class="btn btn-primary btn-lg" data-cursor-hover>
        Get Started Free →
      </button>
    </form>
    <p class="cta-trust">✓ Free 14-day trial &nbsp;·&nbsp; ✓ Cancel anytime &nbsp;·&nbsp; ✓ No credit card</p>
  </div>
</section>
```

- [ ] **Step 2: Add CTA CSS**

```css
/* ── CTA ── */
#cta {
  padding: 140px 0;
  text-align: center;
  position: relative;
  overflow: hidden;
}
.cta-bg-glow {
  position: absolute;
  inset: 0;
  background: radial-gradient(ellipse at center, rgba(99,102,241,0.15) 0%, rgba(245,158,11,0.05) 50%, transparent 70%);
  animation: cta-pulse 4s ease-in-out infinite alternate;
}
@keyframes cta-pulse {
  from { transform: scale(1); opacity: 0.8; }
  to   { transform: scale(1.1); opacity: 1; }
}
.cta-inner { position: relative; z-index: 2; }
.cta-title {
  font-family: var(--font-display);
  font-size: clamp(2.5rem, 5vw, 4rem);
  font-weight: 800;
  line-height: 1.1;
  margin-bottom: 20px;
}
.cta-subtitle {
  font-size: 1.1rem;
  color: var(--muted);
  margin-bottom: 40px;
}
.cta-form {
  display: flex;
  gap: 12px;
  max-width: 520px;
  margin: 0 auto 20px;
  justify-content: center;
}
.cta-input {
  flex: 1;
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: 100px;
  padding: 14px 24px;
  font-size: 0.95rem;
  color: var(--text);
  font-family: var(--font-body);
  outline: none;
  transition: border-color 0.2s, box-shadow 0.2s;
}
.cta-input::placeholder { color: var(--muted); }
.cta-input:focus {
  border-color: var(--primary);
  box-shadow: 0 0 0 3px rgba(99,102,241,0.15);
}
.cta-trust {
  font-size: 0.8rem;
  color: var(--muted);
}
@media (max-width: 540px) {
  .cta-form { flex-direction: column; align-items: center; }
  .cta-input { width: 100%; }
}
```

- [ ] **Step 3: Add CTA scroll entrance animation**

```javascript
// ── CTA SECTION ──
gsap.from('.cta-title, .cta-subtitle, .cta-form, .cta-trust', {
  opacity: 0,
  y: 30,
  duration: 0.8,
  stagger: 0.12,
  ease: 'power3.out',
  scrollTrigger: {
    trigger: '#cta',
    start: 'top 80%',
  }
});
```

- [ ] **Step 4: Verify in browser**

Expected: CTA section with animated gradient glow behind it that pulses. Form elements fade in on scroll. Email input highlights on focus.

---

### Task 13: Footer

**Files:**
- Modify: `index.html` — add footer

- [ ] **Step 1: Add footer markup (after CTA section)**

```html
<!-- Footer -->
<footer id="footer">
  <div class="footer-gradient-line"></div>
  <div class="container">
    <div class="footer-top">
      <div class="footer-brand">
        <div class="nav-logo-text" style="font-family: var(--font-display); font-size:1.4rem; font-weight:800;">Lumina<span style="color:var(--primary)">AI</span></div>
        <p class="footer-tagline">The fastest path to AI mastery.</p>
        <div class="footer-social">
          <a href="#" class="social-icon" data-cursor-hover>𝕏</a>
          <a href="#" class="social-icon" data-cursor-hover>in</a>
          <a href="#" class="social-icon" data-cursor-hover>gh</a>
          <a href="#" class="social-icon" data-cursor-hover>yt</a>
        </div>
      </div>
      <div class="footer-links-group">
        <h4>Product</h4>
        <a href="#">Courses</a>
        <a href="#">Learning Paths</a>
        <a href="#">Certifications</a>
        <a href="#">Enterprise</a>
      </div>
      <div class="footer-links-group">
        <h4>Learn</h4>
        <a href="#">Machine Learning</a>
        <a href="#">Deep Learning</a>
        <a href="#">Data Science</a>
        <a href="#">MLOps</a>
      </div>
      <div class="footer-links-group">
        <h4>Company</h4>
        <a href="#">About</a>
        <a href="#">Blog</a>
        <a href="#">Careers</a>
        <a href="#">Contact</a>
      </div>
      <div class="footer-links-group">
        <h4>Legal</h4>
        <a href="#">Privacy</a>
        <a href="#">Terms</a>
        <a href="#">Cookies</a>
      </div>
    </div>
    <div class="footer-bottom">
      <p>© 2026 Lumina AI. All rights reserved.</p>
    </div>
  </div>
</footer>
```

- [ ] **Step 2: Add footer CSS**

```css
/* ── FOOTER ── */
#footer {
  padding: 80px 0 40px;
  border-top: 1px solid var(--border);
  position: relative;
}
.footer-gradient-line {
  position: absolute;
  top: 0; left: 0; right: 0;
  height: 1px;
  background: linear-gradient(90deg, transparent, var(--primary), var(--accent), transparent);
}
.footer-top {
  display: grid;
  grid-template-columns: 2fr 1fr 1fr 1fr 1fr;
  gap: 48px;
  margin-bottom: 64px;
}
.footer-tagline {
  font-size: 0.875rem;
  color: var(--muted);
  margin: 8px 0 20px;
  line-height: 1.6;
}
.footer-social {
  display: flex;
  gap: 12px;
}
.social-icon {
  width: 36px; height: 36px;
  border: 1px solid var(--border);
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 0.8rem;
  color: var(--muted);
  transition: border-color 0.2s, color 0.2s;
}
.social-icon:hover { border-color: var(--primary); color: var(--primary-light); }
.footer-links-group { display: flex; flex-direction: column; gap: 12px; }
.footer-links-group h4 {
  font-family: var(--font-display);
  font-size: 0.875rem;
  font-weight: 700;
  color: var(--text);
  margin-bottom: 4px;
}
.footer-links-group a {
  font-size: 0.875rem;
  color: var(--muted);
  transition: color 0.2s;
}
.footer-links-group a:hover { color: var(--text); }
.footer-bottom {
  border-top: 1px solid var(--border);
  padding-top: 24px;
  font-size: 0.8rem;
  color: var(--muted);
}
@media (max-width: 900px) {
  .footer-top { grid-template-columns: 1fr 1fr; }
  .footer-brand { grid-column: span 2; }
}
@media (max-width: 480px) {
  .footer-top { grid-template-columns: 1fr 1fr; }
  .footer-brand { grid-column: span 2; }
}
```

- [ ] **Step 3: Add footer fade-in JS**

```javascript
// ── FOOTER ──
gsap.from('#footer .footer-top > *', {
  opacity: 0,
  y: 20,
  duration: 0.7,
  stagger: 0.1,
  ease: 'power2.out',
  scrollTrigger: {
    trigger: '#footer',
    start: 'top 85%',
  }
});
```

- [ ] **Step 4: Verify in browser**

Expected: footer with gradient top border (indigo → amber). 5-column grid on desktop, 2-column on mobile.

---

### Task 14: Lenis Smooth Scroll + Interactive Gradient Background

**Files:**
- Modify: `index.html` — wire up Lenis and scroll-based hue shift

- [ ] **Step 1: Add Lenis initialization and scroll-based gradient (append inside `<script>` block)**

```javascript
// ── LENIS SMOOTH SCROLL ──
const lenis = new Lenis({
  duration: 1.2,
  easing: t => Math.min(1, 1.001 - Math.pow(2, -10 * t)),
  smoothWheel: true,
});

lenis.on('scroll', ScrollTrigger.update);

gsap.ticker.add(time => lenis.raf(time * 1000));
gsap.ticker.lagSmoothing(0);

// ── SCROLL-BASED HUE SHIFT ──
lenis.on('scroll', ({ progress }) => {
  const hue = progress * 30; // 0deg → 30deg hue shift
  document.documentElement.style.setProperty(
    '--scroll-hue-filter',
    `hue-rotate(${hue}deg)`
  );
});
```

- [ ] **Step 2: Add hue-shift overlay CSS (append inside `<style>`)**

```css
/* ── HUE SHIFT OVERLAY ── */
body::before {
  content: '';
  position: fixed;
  inset: 0;
  background: radial-gradient(ellipse at 20% 50%, rgba(99,102,241,0.03) 0%, transparent 60%),
              radial-gradient(ellipse at 80% 20%, rgba(245,158,11,0.02) 0%, transparent 60%);
  pointer-events: none;
  z-index: 0;
  filter: var(--scroll-hue-filter, hue-rotate(0deg));
  transition: filter 0.1s;
}
```

- [ ] **Step 3: Verify in browser**

Expected: scrolling feels buttery and physics-based instead of instant. Subtle color temperature shift as you scroll toward the bottom of the page.

---

### Task 15: Reduced Motion + Final Polish

**Files:**
- Modify: `index.html` — accessibility, final touches

- [ ] **Step 1: Add reduced-motion media query (append inside `<style>`)**

```css
/* ── REDUCED MOTION ── */
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
  .hero-line, #hero-badge, #hero-sub, .hero-cta-row, .hero-trust,
  .hero-scroll-indicator, .course-card, .testimonial-card, .timeline-step {
    opacity: 1 !important;
    transform: none !important;
  }
  .scroll-chevron { animation: none; }
}
```

- [ ] **Step 2: Add selection color and scrollbar styling**

```css
/* ── POLISH ── */
::selection { background: rgba(99,102,241,0.3); color: var(--text); }
::-webkit-scrollbar { width: 6px; }
::-webkit-scrollbar-track { background: var(--bg); }
::-webkit-scrollbar-thumb { background: var(--border); border-radius: 3px; }
::-webkit-scrollbar-thumb:hover { background: var(--primary); }
```

- [ ] **Step 3: Full end-to-end visual QA**

Open `index.html` in browser. Check each item:

- [ ] Page loads with dark background, no FOUC (flash of unstyled content)
- [ ] Fonts load: Space Grotesk for headlines, Inter for body
- [ ] Custom cursor dot + ring tracks mouse
- [ ] Nav fades in on load; becomes glassmorphism on scroll
- [ ] Particle canvas animates in hero background
- [ ] Hero badge → headline lines → subtext → CTA → trust text animate in sequence
- [ ] Hero CTA button has magnetic pull on hover
- [ ] Scrolling down: hero parallax shifts content upward
- [ ] Stats section: numbers count up when scrolled into view
- [ ] Courses section: 6 cards stagger-reveal on scroll, hover lifts + glows
- [ ] How It Works: line draws, steps fade up
- [ ] Testimonials: cards fade in, subtle parallax at different speeds
- [ ] CTA section: pulsing glow, form elements animate in
- [ ] Footer: gradient border line (indigo → amber), links present
- [ ] Smooth scroll throughout (Lenis)
- [ ] No console errors

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: build Lumina AI landing page with GSAP animations"
```

---

## Self-Review

**Spec coverage check:**
- ✅ Nav (sticky glassmorphism) — Task 3
- ✅ Hero cinematic (particles, badge, headline, CTA, parallax) — Tasks 4–7
- ✅ Stats bar with counter — Task 8
- ✅ Feature cards (6 courses, staggered reveal, hover shimmer) — Task 9
- ✅ How It Works timeline (line draw) — Task 10
- ✅ Testimonials (parallax depth) — Task 11
- ✅ CTA banner (pulsing glow, email form) — Task 12
- ✅ Footer (gradient border line) — Task 13
- ✅ Lenis smooth scroll — Task 14
- ✅ Custom cursor — Task 2
- ✅ Magnetic button — Task 7
- ✅ Interactive gradient bg hue shift — Task 14
- ✅ Reduced motion — Task 15

**No placeholders found.** All code blocks complete and self-contained.

**Type consistency:** All JS references match HTML IDs/classes defined in same file. No mismatches detected.
