# Lumina AI — Landing Page Design Spec
_Date: 2026-05-28_

## Overview

A single-file, premium landing page for **Lumina AI**, an online learning platform for AI and data courses. The page targets aspiring data scientists and AI engineers and communicates premium quality through cinematic animations, deep color work, and meticulous micro-interactions.

## Tech Stack

- **Delivery:** Single `index.html` file — no build step, open directly in browser
- **Animations:** GSAP 3 + ScrollTrigger plugin (CDN)
- **Smooth scroll:** Lenis (CDN)
- **Fonts:** Space Grotesk (display) + Inter (body) via Google Fonts
- **Particles:** HTML5 Canvas API (vanilla JS, no library)
- **No frameworks, no build tools, no dependencies to install**

## Color Palette

| Role | Value |
|------|-------|
| Background | `#020617` (near-black navy) |
| Surface | `#0f172a` (card backgrounds) |
| Border | `#1e293b` |
| Primary | `#6366f1` (indigo) |
| Primary Light | `#818cf8` |
| Accent | `#f59e0b` (amber) |
| Text Primary | `#f1f5f9` |
| Text Muted | `#64748b` |
| Gradient | `linear-gradient(135deg, #6366f1, #f59e0b)` |

## Typography

- **Display font:** Space Grotesk — used for all headings, bold and expressive
- **Body font:** Inter — used for body text, captions, and UI labels
- Hero headline: `clamp(3rem, 8vw, 6rem)`, weight 800
- Section headings: `clamp(2rem, 4vw, 3rem)`, weight 700

## Sections

### 1. Navigation (Sticky)
- Logo: "Lumina AI" wordmark with indigo dot
- Links: Courses, Features, Testimonials, Pricing
- CTA button: "Start Learning" with gradient background
- Behavior: starts transparent, gains glassmorphism blur + border on scroll
- Fade-in on page load (GSAP, 0.8s)

### 2. Hero (Cinematic)
- Full viewport height
- **Background:** HTML5 Canvas particle field — 80 floating dots connected by faint lines, slow drift animation
- **Badge:** Pill label "✦ 50,000+ Students Enrolled" with indigo border, fades in first
- **Headline:** "Master AI &" on line 1, "Data Science" on line 2 — gradient text (indigo → amber), staggered word reveal
- **Subtext:** One-line value prop, fades in after headline
- **CTA:** Single "Start Learning Free →" button with magnetic hover effect
- **Trust line:** "No credit card · 14-day free trial · Cancel anytime"
- **Parallax:** Canvas bg moves at 0.3× scroll speed via GSAP
- **Scroll indicator:** Animated chevron at bottom

### 3. Stats Bar
- 4 stats: `50K+ Students` · `200+ Courses` · `95% Completion Rate` · `4.9★ Rating`
- Numbers count up from 0 when scrolled into view (GSAP counter animation)
- Subtle indigo glow separator between each stat
- Glassmorphism card background

### 4. Feature Courses (Cards Grid)
- Section heading: "What You'll Master"
- 6 course cards in a responsive 3×2 grid:
  1. Machine Learning Fundamentals
  2. Deep Learning & Neural Networks
  3. Data Science with Python
  4. Natural Language Processing
  5. Computer Vision
  6. Data Engineering & MLOps
- Each card: course icon (emoji/SVG), title, short description, difficulty badge, "Explore Course →" link
- **Animation:** Staggered scroll-triggered reveal — cards fade in + scale from 0.85 → 1 with 0.1s stagger
- **Hover:** Card lifts 8px, indigo glow border appears, inner shimmer sweeps left→right

### 5. How It Works (Timeline)
- Section heading: "Your Learning Journey"
- 3 horizontal steps: `01 Choose Your Path` → `02 Learn by Doing` → `03 Get Certified`
- Connecting line draws from left to right as section enters viewport
- Each step icon pulses into view with a bounce easing

### 6. Testimonials
- Section heading: "Loved by Learners"
- 3 testimonial cards side by side
- Each: avatar circle (gradient), name, role, star rating, quote
- **Parallax:** cards shift at slightly different Y speeds (0.9×, 1×, 1.1×) as user scrolls
- Fade-in with slight Y offset on scroll enter

### 7. CTA Banner
- Full-width dark section with indigo → amber gradient overlay
- Large headline: "Start Your AI Journey Today"
- Email input + "Get Started Free" button side by side
- Subtle pulsing glow ring behind the section
- Background gradient slowly shifts hue (CSS keyframe animation)

### 8. Footer
- 4-column link grid: Product, Courses, Company, Legal
- Logo + tagline on left
- Social icons row
- Top border: 1px gradient line (indigo → amber)
- Copyright line

## Global Interactions

### Custom Cursor
- Small filled dot (8px, indigo) that tracks mouse exactly
- Larger ring (32px, semi-transparent) that follows with slight lag (lerp)
- On hover over links/buttons: dot disappears, ring scales to 48px and fills slightly
- Hidden on mobile

### Smooth Scroll
- Lenis library wraps the entire page — buttery, physics-based scroll feel
- ScrollTrigger syncs to Lenis's virtual scroll position

### Interactive Background Gradient
- CSS custom property `--scroll-hue` updated on scroll via JS
- Subtle hue rotation of 0° → 30° as user scrolls full page
- Applied to body background as a faint tint overlay

### Button Magnetic Effect
- Hero CTA button: on mouse proximity within 80px, button pulls toward cursor slightly (GSAP QuickTo)
- On mouse leave, snaps back with elastic easing

## Responsiveness
- Fully responsive: mobile (1 col), tablet (2 col cards), desktop (3 col cards)
- Custom cursor disabled on touch devices
- Reduced-motion: `@media (prefers-reduced-motion)` disables parallax and complex animations, keeps fade-ins only

## File Structure
```
index.html   ← entire page: HTML + embedded CSS + embedded JS
```
