# DevNinja Presentations Site — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a static Cloudflare-hosted gallery site at presentations.devninja.in with searchable presentation cards and a self-contained first presentation.

**Architecture:** Root `index.html` fetches `presentations.json` at runtime, renders cards client-side, and filters live on search input. Each presentation lives in its own top-level slug folder as a self-contained HTML file. No build step, no framework.

**Tech Stack:** Vanilla HTML/CSS/JS, Inter font via Google Fonts, Cloudflare Pages (wrangler.toml)

---

## File Map

| File | Role |
|------|------|
| `presentations.json` | Metadata registry — source of truth for all cards |
| `assets/css/style.css` | DevNinja design tokens + gallery component styles |
| `index.html` | Gallery page: header, search bar, card grid, footer, fetch + render logic |
| `three-types-of-confidence/index.html` | First presentation (fully self-contained) |
| `three-types-of-confidence/images/.gitkeep` | Placeholder so images/ directory is tracked |
| `wrangler.toml` | Cloudflare Pages deployment config |
| `_headers` | HTTP cache-control rules |
| `.gitignore` | Excludes node_modules and OS files |

---

## Task 1: Initialize repository

**Files:**
- Create: `.gitignore`

- [ ] **Step 1: Init git repo**

```bash
cd /Users/abhiskum/work/code/learning/devninja/devninja-presentations
git init
```

Expected: `Initialized empty Git repository`

- [ ] **Step 2: Create .gitignore**

```
node_modules/
.DS_Store
.wrangler/
dist/
```

Save to `.gitignore`.

- [ ] **Step 3: Commit**

```bash
git add .gitignore
git commit -m "chore: init repo"
```

---

## Task 2: Create presentations.json registry

**Files:**
- Create: `presentations.json`

- [ ] **Step 1: Create the file**

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

Save to `presentations.json`.

- [ ] **Step 2: Validate JSON is parseable**

```bash
node -e "JSON.parse(require('fs').readFileSync('presentations.json','utf8')); console.log('OK')"
```

Expected: `OK`

- [ ] **Step 3: Commit**

```bash
git add presentations.json
git commit -m "feat: add presentations registry"
```

---

## Task 3: Create shared CSS (devninja design tokens + gallery styles)

**Files:**
- Create: `assets/css/style.css`

- [ ] **Step 1: Create directory and file**

```bash
mkdir -p assets/css
```

- [ ] **Step 2: Write style.css**

```css
:root {
  --tc-green: #00d084;
  --tc-blue: #0070f3;
  --text-primary: #111111;
  --text-secondary: #666666;
  --text-muted: #888888;
  --bg-primary: #ffffff;
  --bg-secondary: #fafafa;
  --border: #e5e5e5;
  --shadow-sm: 0 1px 3px rgba(0,0,0,.08);
  --shadow-md: 0 4px 12px rgba(0,0,0,.1);
  --radius-md: 12px;
  --radius-full: 9999px;
}

*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

body {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
  background: var(--bg-primary);
  color: var(--text-primary);
  -webkit-font-smoothing: antialiased;
}

/* ── Header ── */
.header {
  border-bottom: 1px solid var(--border);
  background: var(--bg-primary);
  position: sticky;
  top: 0;
  z-index: 10;
}
.header-inner {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 24px;
  height: 60px;
  display: flex;
  align-items: center;
  gap: 10px;
}
.logo {
  font-weight: 800;
  font-size: 20px;
  color: var(--text-primary);
  text-decoration: none;
  letter-spacing: -.02em;
}
.logo:hover { color: var(--tc-green); }
.logo-sep { color: var(--border); font-weight: 300; font-size: 20px; }
.site-name { font-size: 14px; font-weight: 500; color: var(--text-muted); }

/* ── Main ── */
.main {
  max-width: 1200px;
  margin: 0 auto;
  padding: 40px 24px 80px;
}

/* ── Search ── */
.search-wrap { margin-bottom: 40px; }
.search-input {
  width: 100%;
  padding: 14px 18px;
  border: 1.5px solid var(--border);
  border-radius: var(--radius-md);
  font-family: inherit;
  font-size: 16px;
  color: var(--text-primary);
  background: var(--bg-primary);
  outline: none;
  transition: border-color .15s;
}
.search-input::placeholder { color: var(--text-muted); }
.search-input:focus { border-color: var(--tc-green); }

/* ── Card grid ── */
.card-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 24px;
}

/* ── Card ── */
.card {
  background: var(--bg-primary);
  border: 1px solid var(--border);
  border-radius: var(--radius-md);
  box-shadow: var(--shadow-sm);
  padding: 24px;
  text-decoration: none;
  color: inherit;
  display: flex;
  flex-direction: column;
  gap: 12px;
  transition: box-shadow .2s, border-color .2s, transform .15s;
}
.card:hover {
  box-shadow: var(--shadow-md);
  border-color: var(--tc-green);
  transform: translateY(-2px);
}
.card-title {
  font-size: 18px;
  font-weight: 700;
  line-height: 1.3;
  color: var(--text-primary);
}
.card-desc {
  font-size: 14px;
  color: var(--text-secondary);
  line-height: 1.6;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}
.card-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
  margin-top: auto;
}
.tag {
  font-size: 12px;
  font-weight: 500;
  color: var(--text-secondary);
  background: var(--bg-secondary);
  border: 1px solid var(--border);
  padding: 3px 10px;
  border-radius: var(--radius-full);
}
.card-date {
  font-size: 12px;
  color: var(--text-muted);
}

/* ── Empty state ── */
.empty {
  display: none;
  text-align: center;
  padding: 80px 0;
  color: var(--text-muted);
  font-size: 16px;
}
.empty.visible { display: block; }

/* ── Footer ── */
.footer { border-top: 1px solid var(--border); background: var(--bg-secondary); }
.footer-inner {
  max-width: 1200px;
  margin: 0 auto;
  padding: 24px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 13px;
  color: var(--text-muted);
}
.footer-link {
  color: var(--tc-blue);
  text-decoration: none;
  font-weight: 500;
}
.footer-link:hover { text-decoration: underline; }

/* ── Responsive ── */
@media (max-width: 768px) {
  .card-grid { grid-template-columns: repeat(2, 1fr); }
}
@media (max-width: 480px) {
  .card-grid { grid-template-columns: 1fr; }
  .main { padding: 24px 16px 60px; }
  .header-inner { padding: 0 16px; }
  .footer-inner { flex-direction: column; gap: 12px; text-align: center; }
}
```

Save to `assets/css/style.css`.

- [ ] **Step 3: Commit**

```bash
git add assets/css/style.css
git commit -m "feat: add devninja design tokens and gallery styles"
```

---

## Task 4: Create gallery index.html

**Files:**
- Create: `index.html`

- [ ] **Step 1: Write index.html**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Presentations · DevNinja</title>
  <meta name="description" content="Technical presentations from DevNinja on AI, engineering, and modern software development.">
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&display=swap" rel="stylesheet">
  <link rel="stylesheet" href="assets/css/style.css">
</head>
<body>

<header class="header">
  <div class="header-inner">
    <a href="https://devninja.in" class="logo">DevNinja</a>
    <span class="logo-sep">/</span>
    <span class="site-name">Presentations</span>
  </div>
</header>

<main class="main">
  <div class="search-wrap">
    <input
      class="search-input"
      type="search"
      id="search"
      placeholder="Search presentations…"
      autocomplete="off"
      aria-label="Search presentations"
    >
  </div>
  <div class="card-grid" id="grid"></div>
  <div class="empty" id="empty">No presentations found.</div>
</main>

<footer class="footer">
  <div class="footer-inner">
    <span>© 2026 DevNinja. Professional tech insights for modern teams.</span>
    <a
      class="footer-link"
      href="https://whatsapp.com/channel/0029Va4bjZE3wtb3J2FWPr2V"
      target="_blank"
      rel="noopener noreferrer"
    >📱 Join Community</a>
  </div>
</footer>

<script>
  const grid = document.getElementById('grid');
  const empty = document.getElementById('empty');
  const search = document.getElementById('search');
  let all = [];

  function formatDate(iso) {
    return new Date(iso + 'T00:00:00').toLocaleDateString('en-US', {
      year: 'numeric', month: 'long', day: 'numeric'
    });
  }

  function render(items) {
    grid.innerHTML = items.map(p => `
      <a class="card" href="/${p.slug}/">
        <div class="card-title">${p.title}</div>
        <div class="card-desc">${p.description}</div>
        <div class="card-tags">${p.tags.map(t => `<span class="tag">${t}</span>`).join('')}</div>
        <div class="card-date">${formatDate(p.date)}</div>
      </a>
    `).join('');
    empty.classList.toggle('visible', items.length === 0);
  }

  function filter(q) {
    const t = q.toLowerCase().trim();
    if (!t) return all;
    return all.filter(p =>
      p.title.toLowerCase().includes(t) ||
      p.description.toLowerCase().includes(t) ||
      p.tags.some(tag => tag.toLowerCase().includes(t))
    );
  }

  search.addEventListener('input', e => render(filter(e.target.value)));

  fetch('presentations.json')
    .then(r => r.json())
    .then(data => {
      all = data.sort((a, b) => b.date.localeCompare(a.date));
      render(all);
    });
</script>

</body>
</html>
```

Save to `index.html`.

- [ ] **Step 2: Verify locally — start a server and open the page**

```bash
python3 -m http.server 8080
```

Open `http://localhost:8080` in a browser.

Check:
- DevNinja logo appears in header, links to devninja.in
- One card renders with title "The Three Types of Confidence"
- Tags "AI", "Career", "Mindset" appear as pills
- Date shows as "May 7, 2026"

- [ ] **Step 3: Verify search works**

Type "rented" in the search bar.
Expected: card disappears (description doesn't contain "rented"), empty state shows.

Type "AI" in the search bar.
Expected: card reappears (matches tag).

Clear the input.
Expected: card reappears.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add gallery page with search"
```

---

## Task 5: Add first presentation

**Files:**
- Create: `three-types-of-confidence/index.html`
- Create: `three-types-of-confidence/images/.gitkeep`

- [ ] **Step 1: Create the folder and placeholder**

```bash
mkdir -p three-types-of-confidence/images
touch three-types-of-confidence/images/.gitkeep
```

- [ ] **Step 2: Create three-types-of-confidence/index.html**

Copy the full presentation HTML (provided by user — 11-slide deck with WebGL animated backgrounds, Red Hat font family, Motion.dev animations) into `three-types-of-confidence/index.html`.

The file starts with:
```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>The Three Types of Confidence · In the AI Era</title>
...
```

The complete file content is the 11-slide presentation HTML shared during design, ending with the Motion.dev animation script block.

- [ ] **Step 3: Verify presentation loads**

With the local server still running (`python3 -m http.server 8080`), open:
`http://localhost:8080/three-types-of-confidence/`

Check:
- Slide 1 renders with animated dark WebGL background
- "Navigate with arrow keys" hint is visible
- Arrow keys advance/retreat slides
- Dot navigation at the bottom works
- All 11 slides navigate correctly

- [ ] **Step 4: Verify gallery card links to presentation**

Open `http://localhost:8080`, click the card.
Expected: navigates to the presentation (slide 1 visible).

- [ ] **Step 5: Commit**

```bash
git add three-types-of-confidence/
git commit -m "feat: add three-types-of-confidence presentation"
```

---

## Task 6: Add Cloudflare deployment config

**Files:**
- Create: `wrangler.toml`
- Create: `_headers`

- [ ] **Step 1: Create wrangler.toml**

```toml
name = "devninja-presentations"
compatibility_date = "2024-01-01"

[site]
bucket = "."
```

Save to `wrangler.toml`.

- [ ] **Step 2: Create _headers**

```
/*/index.html
  Cache-Control: public, max-age=3600

/*.json
  Cache-Control: public, max-age=300
```

Save to `_headers`.

- [ ] **Step 3: Commit**

```bash
git add wrangler.toml _headers
git commit -m "chore: add cloudflare deployment config"
```

---

## Task 7: Final verification

- [ ] **Step 1: Check all files are committed**

```bash
git status
```

Expected: `nothing to commit, working tree clean`

- [ ] **Step 2: Verify full structure**

```bash
find . -not -path './.git/*' -not -path './node_modules/*' | sort
```

Expected output (roughly):
```
.
./.gitignore
./_headers
./assets
./assets/css
./assets/css/style.css
./docs
./docs/superpowers
./docs/superpowers/plans/2026-05-07-devninja-presentations.md
./docs/superpowers/specs/2026-05-07-devninja-presentations-design.md
./index.html
./presentations.json
./three-types-of-confidence
./three-types-of-confidence/images
./three-types-of-confidence/images/.gitkeep
./three-types-of-confidence/index.html
./wrangler.toml
```

- [ ] **Step 3: Verify responsive layout**

Open `http://localhost:8080` and resize browser to mobile width (< 480px).
Expected: cards stack to 1 column, header and footer padding reduces.

- [ ] **Step 4: Verify empty state**

Type a search term that matches nothing (e.g., "zzzzz").
Expected: grid is empty, "No presentations found." message is centred.

- [ ] **Step 5: Done**

Site is ready to deploy. Point Cloudflare Pages to this repo, set build command to none, output directory to `/`.
