# DeepSeek 4.1 Flash 1017 — Concept Landing Page

A static, single-page concept site for a fictional low-latency inference model.
The page is built as a **deep-sea photo essay**: full-bleed colour chapters, real NOAA deep-sea photography, and a liquid-glass interface layer on top.

**Live demo:** <https://ds-4-1-flash-1017-web.pages.dev>

> This is an independent design concept, not an official DeepSeek release. All performance numbers, prices and dates are illustrative placeholders.


---

## What it is

- **One static HTML file** (`public/index.html`, inline CSS + JS, no build step) plus 5 photographs.
- **A marketing argument, not a template.** The product’s name is a collision: *DeepSeek* (the deep sea, the whale) and *Flash* (a burst of light). The page turns that into a sequence of exposures — paper white → black water → electric blue → deep-sea red → photographs → a lit request card → a giant wordmark.
- **A working live thing:** the “一次请求 / One request” card. Four real controls (task, context window, streaming, thinking depth) rewrite the response card instantly.
- Designed and tested at 1440 / 768 / 390 px; each breakpoint is its own composition.

## Sections

| # | Section | Form |
| --- | --- | --- |
| 1 | Hero — *Thinking shouldn’t wait / The flash is already here* | Paper white + giant liquid-glass blue orb |
| 2 | Why “Flash” — the name | Black mosaic of colour tiles (blue / white / photo) |
| 3 | Wait calculator — *How much waiting do you save?* | Electric blue field + frosted calculator |
| 4 | Capability marquee | Deep-sea red band, names running off both edges |
| 5 | Deep-sea photo essay — *The light here is their own* | 3 real ROV photographs + captions |
| 6 | Capability plate | White 50/50 split: square photo + tabs + two actions |
| 7 | One request (interactive) | Black field, pinned frosted card + live controls |
| 8 | Price | One giant sentence with a blue price block |
| 9 | FAQ | Blue heading band + alternating black/white bands |
| 10 | Apply — *Write your email into ___ and get in* | Sentence-shaped form (mailto, no backend) |
| 11 | Footer | Giant DEEPSEEK wordmark knocked out of white blocks |

## Interactions

- **Wait calculator** — `calls/day × 1.66 s ÷ 3600` updates a huge hours figure; one-time count-up on first view.
- **Request card** — task (reasoning / code / long-form / multilingual), context (8K / 32K / 128K / 1M), streaming (on/off), thinking depth (fast / deep); the card’s answer, code block and metadata change accordingly.
- **Capability plate** — three tabs switch title + description (arrow keys supported).
- **Application form** — validates the email locally, then opens a `mailto:` draft (works with no backend).
- **Floating glass toolbar** — appears when scrolling up, hides when scrolling down; never overlaps content while scrolling down.
- Full `prefers-reduced-motion` support; no JavaScript required to read the page.

## Design notes

- Direction: **colour chapters** (from `design-taste-frontend` v2, seed `257652249`). Every section is one flat field floor-to-ceiling; the same 12-column grid and margins run through all of them.
- Palette taken from deep-sea photography: black water, floodlight white, electric blue `#2436ff`, deep-sea red `#ff3b1f`. The red is a real phenomenon — red light never reaches that depth.
- Type: **Noto Sans SC** 400/700/900 for Chinese, **Red Hat Display 900** for Latin display and numerals. Seven sizes total (15 / 18 / 24 / 32 / 44 / 107 / 274 px).
- **Liquid glass layer** (requested by the client): the hero orb, floating toolbar, version chip, request card, calculator panel, buttons and the closing input. This deliberately overrides the base direction’s “no gradients / no shadows / no glass” rules — it is scoped to UI surfaces only; the colour chapters stay flat.
- Photography is real ROV / submersible imagery from NOAA and partners, all public domain. One grade is applied to every image (slightly darker, slightly desaturated) so the set reads as one light source.

## Tech

- Static HTML + inline CSS + vanilla JS. No framework, no bundler, no server.
- 5 JPEG photographs + 2 text assets; **1.68 MB total**.
- Google Fonts (Noto Sans SC, Red Hat Display) with a system-font fallback — the layout survives without network access.
- `public/_headers` adds security headers and immutable caching for `/assets/*`; `public/robots.txt` allows indexing.

## Repository layout

```
public/                 ← the deployed site (Cloudflare Pages output directory)
  index.html
  _headers
  robots.txt
  assets/img/*.jpg
  assets/manifest.json  ← machine-readable image credits
  assets/CREDITS.md
README.md               ← this file
README.zh-CN.md         ← Chinese design notes
COPY.md                 ← copy deck (Chinese)
DEPLOY.md               ← deployment notes (Chinese)
```

## Local preview

Open `public/index.html` directly in a browser. Nothing needs installing.

## Deploy

Cloudflare Pages, **Direct Upload** (Cloudflare’s 2026 dashboard turns Git imports into Workers; this project is deployed as a real Pages project via the official CLI):

```powershell
npx --yes wrangler@4 pages deploy public `
  --project-name=ds-4-1-flash-1017-web `
  --branch=main --commit-dirty=true
```

First run on a new machine: `npx --yes wrangler@4 login` and approve in the browser.
The site is source-controlled in `public/`; pushing to GitHub does not redeploy automatically (Direct Upload), so run the command above after changes.

## Verification

- Source lint (`lint.mjs`): **0 FAIL / 6 WARN** — the WARNs are the `backdrop-filter` declarations of the requested liquid-glass layer.
- Browser inspection (`inspect.mjs`, 1440/768/390, run with `--allow gradient,shadow,glass,button-fx`): **0 FAIL / 1 WARN** — the only warning is “no video on the page”; no clean, licence-safe deep-sea clip could be obtained (available clips contain burned-in captions or ROV branding).
- Live check (Edge headless, production URL): HTTP 200, correct title, 5/5 images served, 26 requests, 0 non-200, 0 failures, 0 horizontal overflow at 1440 and 390, fonts loaded.
- `_headers` confirmed on the live site (`X-Content-Type-Options: nosniff`, `Referrer-Policy`, immutable image caching).

## Photographs & credits

All five photographs are **public domain**, downloaded from Wikimedia Commons.

| File | Subject | Credit |
| --- | --- | --- |
| `assets/img/01-halitrephes.jpg` | *Halitrephes maasi* medusa, ~1,200 m | NOAA Office of Ocean Exploration and Research |
| `assets/img/02-benthocodon-hyalinus.jpg` | *Benthocodon hyalinus*, Chukchi Borderlands | Caitlin Bailey, GFOE / NOAA OER |
| `assets/img/02-marrus.jpg` | *Marrus orthocanna* siphonophore | Kevin Raskoff, Monterey Peninsula College |
| `assets/img/03-dumbo-square.jpg` | Dumbo octopus (*Grimpoteuthis*) | NOAA Okeanos Explorer |
| `assets/img/06-coral.jpg` | Coral garden, Sibelius Seamount, 2,465 m | NOAA OER / Musicians Seamounts expedition |

Source pages and licence records: [`public/assets/CREDITS.md`](public/assets/CREDITS.md) and [`public/assets/manifest.json`](public/assets/manifest.json). Credits are also printed in the site footer.

## Disclaimer

Independent concept page. Not affiliated with or endorsed by DeepSeek. “DeepSeek”, “4.1 Flash” and “1017” are used to describe a fictional product; all figures, prices, release dates and beta terms on the page are placeholders for demonstration.
