# Econify Zine — Opening Experience Design
**Date:** 2026-05-09  
**Status:** Approved

---

## Overview

Adds a cinematic page-load experience and three ambient interaction enhancements to the existing `index.html`. Pure CSS + vanilla JS, no external libraries.

---

## 1. Opening Animation: Ink Bleed

**What it does:** A full-screen overlay greets the visitor. The screen starts white with the ECONIFY ZINE title sitting in dark ink at the center. A radial ink blot expands from the center outward — slow then sudden — flooding the screen dark. The title becomes a white cutout in the darkness, a red rule animates in beneath it. Then the overlay dissolves and the real page appears.

**Phases:**

| Phase | Duration | What happens |
|---|---|---|
| 0 | 0–0.3s | Overlay is white; title visible in `--ink` colour |
| 1 | 0.3–1.4s | Dark radial-gradient expands via `clip-path: circle()` from 0% → 120% |
| 2 | 1.4–2.0s | Title switches to white; red underline rule sweeps in left→right |
| 3 | 2.0–2.6s | Overlay fades to opacity 0 and is removed from DOM |
| 4 | 2.6s | Page content visible; scroll reveals begin as normal |

**Implementation:**
- `<div id="ez-intro">` injected at top of `<body>`, `position:fixed`, `z-index:9999`
- Ink spread: `clip-path: circle(0% at 50% 50%)` → `circle(150% at 50% 50%)` via CSS keyframes + `cubic-bezier(0.22, 1, 0.36, 1)`
- Title colour flip: CSS class toggle at phase 2 (`--ink` → `white`)
- Red rule: `width: 0 → 6rem` transition
- Skip button: `position:absolute; top:1.5rem; right:2rem`, appears after 1s delay, advances to phase 3 immediately on click
- Session skip: checked/set in `sessionStorage` — replay on hard refresh, skip on soft navigation
- `prefers-reduced-motion`: skip entire intro, show page immediately

---

## 2. Film Grain

**What it does:** A subtle animated noise texture sits over the entire page at low opacity, giving the site a printed-paper tactile quality.

**Implementation:**
- SVG `<filter id="ez-noise">` with `<feTurbulence type="fractalNoise" baseFrequency="0.65" numOctaves="3" stitchTiles="stitch">` injected once into the DOM
- `<div id="ez-grain">` — `position:fixed; inset:0; pointer-events:none; z-index:200; opacity:0.035` — applies the filter via CSS `filter: url(#ez-noise)`
- `@keyframes grain`: randomly offsets `transform: translate()` every frame using `steps(8)` — creates shimmering effect without canvas
- `prefers-reduced-motion`: grain div is not added

---

## 3. 3D Card Tilt

**What it does:** Article cards and the Edition 01 cover image tilt in 3D perspective as the mouse moves over them, with a subtle specular highlight that follows the tilt.

**Targets:** `.article-card` (all article mosaic cards) and `.spotlight-cover img`

**Implementation:**
- JS `mousemove` listener on each target element
- Calculate normalised mouse position within element: `x = (e.offsetX / w - 0.5) * 2`, `y = (e.offsetY / h - 0.5) * 2`
- Apply: `transform: perspective(700px) rotateY(${x * 8}deg) rotateX(${-y * 8}deg) scale(1.02)`
- Specular highlight: `::after` pseudo-element with `radial-gradient` that tracks mouse position via CSS custom properties `--mx` / `--my`
- `mouseleave`: reset transform to identity with `transition: transform 0.5s var(--ease)`
- Mobile / touch: skipped entirely (detected via `matchMedia('(hover: none)')`)
- `prefers-reduced-motion`: skipped

---

## 4. Article Hover Reveals

**What it does:** Hovering an article card triggers a clip-path colour wash from left to right, the card title lifts slightly, and a red top-border appears — like a magazine spread activating.

**Implementation:**
- Each `.article-card` gets a `::before` pseudo-element: `position:absolute; inset:0; background: var(--red-pale)` clipped with `clip-path: inset(0 100% 0 0)` initially
- On `:hover`: clip-path transitions to `inset(0 0% 0 0)` — full-width colour wash — over `0.35s cubic-bezier(0.22,1,0.36,1)`
- Card title: `transform: translateY(0) → translateY(-3px)` on hover
- Top border: `box-shadow: inset 0 3px 0 var(--red)` fades in on hover
- All transitions: `0.3s var(--ease)`

---

## Constraints

- Single file — all changes made to `index.html`
- No external JS libraries or CDN dependencies added
- Total JS added: < 80 lines
- Total CSS added: < 120 lines
- Does not break existing scroll-reveal, nav, newsletter, or drawer functionality
- Passes `prefers-reduced-motion` — all effects disabled cleanly

---

## Out of Scope

- Magnetic cursor (excluded per professional recommendation)
- Extra ticker (site already has one)
- Scroll parallax (mobile compatibility concerns)
