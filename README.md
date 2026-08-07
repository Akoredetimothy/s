# Personal Research Site — N10 × NokiaFlux

A static personal site for a researcher, styled after the **N10** (dark, Windows-Phone-Metro × Nokia/Symbian) and **NokiaFlux** (light, high-contrast) design languages — combined into one shell with a **theme toggle** (dark/light) and an **accent switcher**.

All content is **markdown**. There is **no build step**: a single `index.html` loads markdown files in the browser with [marked.js](https://github.com/markedjs/marked) and routes everything through URL hashes (`#home`, `#books`, `#publications`, `#blog`, `#cv`, `#blog/<slug>`, `#books/<slug>`).

**The blog and books index themselves.** Drop any `.md` file into `blog/` or `books/` and push — the listing pages discover them automatically via the GitHub API. No index file to update.

The whole site can be authored from **Obsidian** — open this folder as a vault, write posts/book notes as plain markdown with Properties (front matter), and publish with one `git push`.

---

## Quick Start (GitHub Pages)

1. Copy **all files** to the root of your GitHub repo
2. Repo **Settings → Pages** → Source: `Deploy from a branch` → `main` / `(root)` → Save
3. Live at `https://<username>.github.io/<repo>/` within a minute or two

> Using `<username>.github.io` (a user site) as the repo name works too — auto-detection handles both.
> **Custom domain?** Set `SITE.owner` / `SITE.repo` in `index.html` (see *Custom domains* below).

No `npm install`, no Actions, no Jekyll (a `.nojekyll` file is included to be sure).

---

## Structure

```
repo/
├── index.html               Single-file shell: all CSS + JS + routing + theme engine
├── content/
│   ├── home.md              About page — photo panel, bio, tiles, links
│   ├── publications.md      Publications — styled .pub-entry cards, grouped by year
│   ├── cv.md                CV — .cv-entry blocks (education, roles, awards…)
│   ├── blog.md              Fallback blog index (manual; used only if auto-discovery fails)
│   └── books.md             Fallback books index (manual; used only if auto-discovery fails)
├── blog/                    Each .md file here becomes a post automatically
├── books/                   Each .md file here becomes a book note automatically
├── images/                  Drop photos/images here; reference as `![](images/x.png)`
├── templates/               Obsidian templates: blog.md, book.md
├── .obsidian/               Obsidian vault config (workspace.json is git-ignored)
├── .nojekyll
└── README.md
```

---

## Authoring from Obsidian

1. In Obsidian: **Open folder as vault** → select this repo folder.
2. Settings → Files & Links → New link format: **Markdown**, Default location for attachments: **`images/`** (already configured in `.obsidian/`).
3. To write a post: create a note in `blog/` (or use the **Insert template** → `templates/blog.md`).
4. To write a book note: create a note in `books/` (template: `templates/book.md`).
5. Drag images into the note — Obsidian drops them into `images/` and writes `![](images/…)` links automatically.
6. Commit and push — the site updates within a minute or two.

> Keep links between notes as standard markdown links (`[text](#blog/slug)`), not Obsidian wikilinks `[[…]]`, so they resolve on the live site.

---

## Adding a blog post (the bit you'll do most)

1. Create `blog/<my-slug>.md` (kebab-case filename):

```markdown
---
title: My New Result
date: 2026-07-24
tags: ml, preprint
excerpt: One sentence shown under the title in the blog index.
---

# My New Result

Write in normal markdown. Headings, tables, code blocks, images…
```

2. Push to GitHub. **Done** — it appears at `#blog` and `#blog/<my-slug>`.

Front matter rules (all fields optional, but recommended):

| Field | If missing |
|---|---|
| `title` | The first `# Heading` in the body is used |
| `date`  | Post is undated and sorts below dated posts |
| `tags`  | No tag chips shown |
| `excerpt` | First paragraph of the body is used |

> The list is cached in your browser tab for 5 minutes, and GitHub Pages itself takes 1–2 minutes to pick up a push — so allow a short delay before a new post is visible.

---

## Editing the pages

### About (`content/home.md`)
Standard markdown, plus a **picture panel**. Replace `images/photo.svg` with your own photo and update the `src` (square-ish images, ≥600×600 px, work best). The panel renders duotone by default and snaps to full color on hover; replace the whole `about-panel` block if you'd rather have plain text.

```html
<div class="about-panel">
  <div class="about-photo">
    <img src="images/photo.jpg" alt="Portrait of Me">
    <div class="sweep"></div>
    <div class="about-photo__caption"><span>FIG.01</span><b>PORTRAIT</b></div>
  </div>
  <div class="about-body">
    <p class="lead"><strong>Your one-liner</strong> …</p>
  </div>
</div>
```

### Publications (`content/publications.md`)
Group years under `## 2026` headings; wrap each paper in a card:

```html
<div class="pub-list">

<div class="pub-entry">
  <div class="pub-title">Paper Title</div>
  <div class="pub-meta"><strong>Your Name</strong>, Coauthor</div>
  <div class="pub-venue">Venue · Year</div>
  <div class="pub-links">
    <a href="paper.pdf" target="_blank">PDF</a>
    <a href="https://doi.org/…" target="_blank">DOI</a>
    <a href="https://github.com/…" target="_blank">Code</a>
  </div>
</div>

</div>
```

Cards auto-number (01, 02, …) and your name renders in the accent color via `<strong>`.

### CV (`content/cv.md`)
Use `.cv-entry` blocks inside headings. Optional: put a PDF at `files/cv.pdf` and keep the download button at the top.

```html
<div class="cv-entry">
  <div class="cv-entry-header">
    <div>
      <div class="cv-entry-title">Role / Degree</div>
      <div class="cv-entry-subtitle">Organization</div>
    </div>
    <div class="cv-entry-date">2024 — Present</div>
  </div>
  <div class="cv-entry-details">Description…</div>
</div>
```

---

## Making it yours — identity

Everything lives in `index.html`; search for these:

| What | Where | Default |
|---|---|---|
| Browser tab title | `<title>` | `Your Name — Research` |
| Logo tile initials | `<span class="face face--front mark">YN</span>` (two places: header + footer `foot-mark`) | `YN` |
| Header name | `<span class="n name">Your Name</span>` | `Your Name` |
| Badge | `<span class="badge">Research</span>` | `Research` |
| Footer name | second `<span class="name">` | `Your Name` |
| Default page note | `ROUTES` object in the JS | e.g. `profile and research interests` |

Design tokens (colors, fonts, radius) are CSS variables at the top of `index.html` — both the dark (N10) and light (NokiaFlux) palettes are defined; users switch between them live with the ⚙ button, and their choice persists via `localStorage`. Accent color (blue/red/green/yellow/violet/rose) is also user-switchable, saved per theme.

---

## How auto-discovery works

- `index.html` works out your repo from the URL: `https://<owner>.github.io/<repo>/…` → `owner` + `repo`
- It calls `api.github.com/repos/<owner>/<repo>/contents/blog` once (cached 5 min per tab) to list `.md` files
- Then fetches each file to read its front matter (title/date/tags/excerpt)
- Post bodies are fetched relative to the site (`blog/<file>.md`), so post rendering works offline from the API and even on `localhost`

**Fallback:** if discovery can't run (you're testing locally, the GitHub API rate-limits you — 60 calls/hour/IP unauthenticated — or the repo can't be detected), the site transparently switches to the manual list in `content/blog.md`, in the format:

```markdown
* [Post Title](#blog/post-slug) — July 24, 2026
```

The blog header shows which mode it's in (`auto-discovered from /blog` vs `manual listing`).

### Custom domains
On a custom domain the URL doesn't reveal your repo, so open `index.html` and fill in:

```js
const SITE = {
  owner: 'yourusername',   // GitHub username or org
  repo:  'your-repo',      // repository name
  branch: 'main',
  ...
};
```

---

## Testing locally

`file://` blocks markdown fetching — run a server instead:

```bash
cd <repo folder>
python3 -m http.server 8080
# → http://localhost:8080
```

Blog index will say `manual listing` locally (that's the fallback; nothing is broken).

---

## Troubleshooting

| Symptom | Fix |
|---|---|
| Page content or posts don't load locally | Serve over HTTP (above) — `file://` is blocked by CORS |
| New post not on the live site yet | Pages deploys take 1–2 min; the blog list is also cached 5 min per tab — close/reopen the tab |
| Blog says `manual listing` on the live site | Check DevTools → Network for the GitHub API call; you may be rate-limited (waits resolve itself) or on a custom domain without `SITE.owner/repo` set |
| Post shows but with no date | Add `date: YYYY-MM-DD` to the front matter |
| Fonts look like generic monospace | First load needs internet (Google Fonts CDN); offline it falls back gracefully |
| Styling looks unstyled | Everything is inline in `index.html` — make sure you copied *all* files as-is |

---

## Credits

- Aesthetic: N10 UI (Metro × Symbian, dark) and NokiaFlux (light high-contrast) — merged into one dual-theme shell
- Fonts: Bitcount Single, Quantico, JetBrains Mono (Google Fonts)
- Markdown: [marked.js](https://github.com/markedjs/marked) via jsDelivr
- Deploy target: GitHub Pages, zero build
