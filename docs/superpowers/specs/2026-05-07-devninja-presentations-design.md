# DevNinja Presentations Site — Design Spec

**Date:** 2026-05-07
**Domain:** presentations.devninja.in
**Deployment:** Cloudflare Pages (static, no build step)

---

## Overview

A static website that hosts HTML slide presentations under `presentations.devninja.in`. The root page is a searchable gallery of presentation cards. Each presentation lives in its own top-level folder and is fully self-contained HTML.

---

## Repository Structure

```
devninja-presentations/
├── index.html                          # Gallery page: search bar + card grid
├── presentations.json                  # Metadata registry for all presentations
├── assets/
│   └── css/style.css                   # Shared styles (devninja design tokens)
├── wrangler.toml                       # Cloudflare deployment config
├── _headers                            # Cache-control headers
└── three-types-of-confidence/
    ├── index.html                      # Presentation HTML (self-contained)
    └── images/                         # Assets for this presentation
```

Each future presentation follows the same pattern: `<slug>/index.html` + `<slug>/images/`.

---

## presentations.json Schema

```json
[
  {
    "slug": "three-types-of-confidence",
    "title": "The Three Types of Confidence",
    "description": "How AI is forcing engineers to examine what kind of confidence they actually own.",
    "tags": ["AI", "Career", "Mindset"],
    "date": "2026-05-07"
  }
]
```

Adding a new presentation = drop the folder + append one object to this file.

---

## Gallery Page (index.html)

### Design System
Matches devninja.in exactly — TechCrunch-inspired light theme:
- **Background:** `#ffffff` / `#fafafa` (card background)
- **Primary accent:** `#00d084` (green)
- **Link color:** `#0070f3` (blue)
- **Text:** `#111111` primary, `#666666` secondary, `#888888` muted
- **Font:** Inter (same as devninja.in)
- **Cards:** white bg, `1px solid` light border, `border-radius: 12px`, subtle box-shadow

### Layout
- **Header:** "DevNinja" logo (links to devninja.in) + "Presentations" page title
- **Search bar:** full-width input below header; filters cards live on `input` event
- **Card grid:** 3 columns (2 on tablet ≤768px, 1 on mobile ≤480px)
- **Footer:** "© 2026 DevNinja" + WhatsApp community link (same as devninja.in)

### Card
Each card shows:
- **Title** — bold, `#111111`
- **Description** — 2-line clamp, `#666666`
- **Tags** — pill chips, muted background
- **Date** — small, `#888888`, formatted as "Month DD, YYYY"

Clicking a card navigates to `/<slug>/` in the same tab.

### Search Behaviour
- Filters against: `title`, `description`, and `tags` (case-insensitive)
- Empty state: "No presentations found" message centred below the search bar
- No server calls — purely client-side, reads the already-fetched JSON

### Data Flow
1. `index.html` fetches `presentations.json` on `DOMContentLoaded`
2. Renders all cards sorted by `date` descending
3. Search input filters the in-memory array and re-renders cards

---

## Presentation Pages

Each presentation is a fully self-contained HTML file with its own styles, scripts, and CDN dependencies. No shared CSS or JS is injected. The gallery links directly to `/<slug>/`.

---

## Cloudflare Deployment

```toml
# wrangler.toml
name = "devninja-presentations"
compatibility_date = "2024-01-01"

[site]
bucket = "."
```

```
# _headers
/*/index.html
  Cache-Control: public, max-age=3600

/*.json
  Cache-Control: public, max-age=300
```

No build command. Deploy root directory as-is.

---

## First Presentation

- **Slug:** `three-types-of-confidence`
- **File:** `three-types-of-confidence/index.html`
- **Content:** 11-slide deck on Rented / Earned / Owned confidence in the AI era
- **Tech:** Self-contained HTML with WebGL animated backgrounds, Red Hat font family, Motion.dev animations from CDN
