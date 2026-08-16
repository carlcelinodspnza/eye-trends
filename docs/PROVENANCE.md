# Provenance

Where this build came from, what in it is original, and what you must check before publishing it.
Read this before deploying to any public-facing address.

## Origin

Every one of the 43 pages was produced by an automated clone of:

```
https://jc1147.github.io/squah-previews/eyetrends/
```

Cloned `2026-08-06T02:29:15.531Z` (recorded in `audit.json` as `emitted_at`). That source is itself a
GitHub Pages preview under the **`jc1147`** account — a third-party account, not one of ours. The
per-page source URL for each file is recorded in `audit.json` under `pages[]`.

What the clone tooling changed relative to the source:

- 5 source stylesheets (215 CSS sources) were consolidated into the single `chrome.css`.
- Internal navigation hrefs were rewritten to point at sibling files in a flat directory.
- Cross-origin font files were mirrored locally into `_xorigin/fonts.gstatic.com/`.
- Analytics and tracking tags (GA, GTM, Facebook pixel, Hotjar and similar) were stripped.
- 172 links were removed (`audit.json` → `transformations.links_removed`).

## The subject is a real, operating business

The site reproduces the identity of a live optometry practice:

| | |
|---|---|
| Business | Eye Trends Vision & Glasses Center (Eye Trends Clear Lake) |
| Practitioner | Dr. Jerry Hyder, OD |
| Address | 515 Bay Area Blvd Ste 300, Houston, TX 77058 |
| Phone | (281) 488-0066 — appears as `tel:+12814880066` throughout the build |
| **Their own live website** | **https://www.eyetrendsclearlake.com/** |

**This matters.** The business already has a website. This build is a second public copy of their
name, practitioner, address, phone number and service descriptions, sitting at a different URL. Two
consequences:

1. **Search.** If indexed, these pages compete with and can be confused for the practice's real
   site. `robots.txt` at the repo root requests that crawlers stay away — but `robots.txt` is a
   request honoured by well-behaved crawlers, not an access control, and it does **not** remove URLs
   that are already known. The stronger control is a per-page
   `<meta name="robots" content="noindex, nofollow">` plus a `<link rel="canonical">` pointing at
   `eyetrendsclearlake.com`. **Neither is currently present on any of the 43 pages.**
2. **Visibility.** This repository is public. Anyone with the URL can read both the rendered pages
   and the source. `robots.txt` does not make anything private.

## Rights

The page content, business name, practitioner name, address, phone number and any photography
originating from the source are **not ours**. This repository carries no licence grant for them.

Before this is published anywhere presented as live, confirm in writing:

- [ ] The practice has authorised the use of their name, likeness and business details.
- [ ] Rights are cleared for every image that came from the source.
- [ ] Whoever built the `jc1147` source preview has agreed to this derivative.
- [ ] Content accuracy has been reviewed by the practice — hours, insurance, and clinical service
      claims in particular. Nothing in this build has been fact-checked against the practice.

## What is original to this build

Three images were supplied by the owner and added on **2026-08-06**, on `index.html` only. Each was
sized to its measured container box:

| File | Container | Size | Pages using it |
|---|---|---|---|
| `assets/real/trust-banner.png` | `.ethc-trust__photo` | 1120 × 210 | 1 |
| `assets/real/dr-hyder-portrait.png` | `.et-doctor__portrait` | 511 × 560 | 1 |
| `assets/real/optical-showroom.png` | `.ethc-close__media` | 455 × 480 | 1 |

These were added as **new files** rather than overwriting the originals they replaced, because those
originals are shared across the site — overwriting would have silently changed dozens of untouched
pages. Current reference counts, verified against the working tree:

| Original | Still referenced on |
|---|---|
| `assets/real/dr-hyder.jpg` | 38 of 43 pages |
| `assets/real/optical-1.jpg` | 23 of 43 pages |
| `assets/real/optical-2.jpg` | 8 of 43 pages |

**The other 42 pages still carry the original source imagery.** See `OPEN-ITEMS.md`.

Container mechanics, if you need to swap these again: `.ethc-trust__photo` is `aspect-ratio`-driven
with `overflow:hidden` + `object-fit:cover`. `.et-doctor__portrait` is `position:absolute` filling
`.et-doctor__media`, whose aspect ratio flips from 4/5 portrait to 16/11 landscape at ≤768px.

## Build receipt

`audit.json` at the repo root is the clone tooling's own record: `shape: "mp"`,
`chrome_css_chars: 222613`, `css_sources_consolidated: 215`, `assets_emitted: 20`, a `pages[43]`
array with each page's source URL, an `internal_link_map`, and a `stage_5_audit` block reporting
12/12 acceptance gates passed with 0 hard and 0 soft issues.

Treat that 12/12 as **narrow**. Those gates do not verify that relative local CSS and image
references actually resolve on disk — a completely unstyled page has scored 12/12 under them before.
The meaningful fidelity evidence is separate: a round-trip pixel diff of each emitted page against
its own live source reference returned 43/43 pages passing across 258/258 viewports, 36 pages at
100.000%, lowest single viewport 99.983% (`book-appointment.html`, a form-heavy page).

Also note `audit.json` records `bundle_root` as `…/Code/clones/jc1147.github.io` — a stale path from
before the directory was renamed to `Eye Trends`. Cosmetic, but don't trust that field.

## History

| Date | Event |
|---|---|
| 2026-08-06 | Cloned, audited, 3 owner images swapped on `index.html`; pushed to `carlcelinodspnza/eye-trends-preview` and served at `carlcelinodspnza.github.io/eye-trends-preview/` |
| 2026-08-17 | Moved to this repo, `carlcelinodspnza/eye-trends`, with history preserved. The old preview repo was left live and untouched so existing links keep working. |
