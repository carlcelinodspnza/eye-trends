# Eye Trends Vision & Glasses Center

A 43-page static website build for Eye Trends Vision & Glasses Center — an optometry practice in
Clear Lake, Houston, TX. Pure HTML and CSS: no build step, no framework, no dependencies, no
JavaScript files.

**Live:** https://carlcelinodspnza.github.io/eye-trends/

> **Status: not a production site.** This is a build artifact derived from an existing preview, and
> it reproduces the identity of a real, operating business that already has its own website. Read
> [`docs/PROVENANCE.md`](docs/PROVENANCE.md) before deploying this anywhere public-facing, and
> [`docs/OPEN-ITEMS.md`](docs/OPEN-ITEMS.md) for the known defects.

## Quick start

It is a static site — open it with any HTTP server. Do **not** open the pages via `file://`; a
directory URL won't resolve to `index.html` and links will look broken when they aren't.

```bash
cd "Eye Trends"
python3 -m http.server 8000
# then visit http://localhost:8000/
```

## How the site is put together

| Thing | Where | Notes |
|---|---|---|
| Pages | 43 `.html` content pages, all at the repo root, plus `404.html` | Flat structure — no nested page directories |
| Styles | `chrome.css` (223,507 bytes) | The **only** stylesheet, consolidated from 4 source sheets. All 43 content pages link it; `404.html` deliberately does not — it carries its own inline styles so it renders when served for a nested missing path. |
| Design tokens | `:root` block at **lines 424–523** of `chrome.css` — where the consolidated `tokens.css` section starts, not the top of the file | 77 custom properties: brand palette, three `font-family` tokens, spacing / z-index / opacity / duration scales, radii, shadows, gradients. **No type scale** — font sizes are hard-coded in the rules. |
| Images | `assets/real/` (7) · `assets/gen/` (5) · `assets/full-library/` (10) | 22 files, plus `assets/favicon.svg` at the `assets/` root — 23 in total. The favicon is the site icon. |
| Fonts | `_xorigin/fonts.gstatic.com/s/` | 10 self-hosted `.woff2`, but only the **7 Inter files** are referenced by relative `url()` in `chrome.css`. The 3 Fraunces files are dead weight — see OPEN-ITEMS #3. |
| Behaviour | Inline `<script>` blocks | Each of the 43 content pages carries two or three (20 have 2, 23 have 3); `404.html` carries none. **Zero external script files** and no analytics or tracking tags. |
| Clone receipt | `audit.json` | Machine record from the tooling that produced the build — see PROVENANCE |

Three properties worth knowing before you touch anything:

- **There are no page-level `<style>` blocks** on any of the 43 content pages. (An earlier
  auto-generated README claimed otherwise; it was wrong — verified 0 across all 43 pages.
  `404.html` does carry one, deliberately, because it must render standalone.) Stylesheet *rules*
  all live in `chrome.css` — but the pages also carry **901 inline `style="…"` attributes**, mostly
  `--et-i` animation-stagger indices, though some are real `object-fit`, `margin`, `color` and
  `font-size` declarations. Check those too before concluding a value comes from the stylesheet.
- **Each of the 43 content pages carries two third-party `<link rel="preconnect">` hints** to
  `fonts.googleapis.com` and `fonts.gstatic.com` (`404.html` carries none). They are inert
  leftovers — no stylesheet is ever fetched from either — but they are outbound connections to
  Google on each page load. See OPEN-ITEMS #1.
- **Every link and asset path is relative.** Zero root-absolute references (`href="/…"` and
  `src="/…"` both count 0; there are 4,502 bare sibling page links and 170 bare `assets/…` sources).
  That means the site serves correctly from **any** subpath, so the repo can be renamed or the site
  moved to a different host without rewriting a single URL.

## Brand tokens

Defined in `:root` in `chrome.css`:

| Token | Value | Role |
|---|---|---|
| `--ds-primary` | `#0B7360` | Optic Teal — brand accent / CTA surface |
| `--ds-primary-deep` | `#0E1C1E` | Near-black cool ink — dark bands, footer, headings |
| `--ds-accent` | `#E0A44C` | Warm Amber — decorative pop |
| `--ds-bg-surface` | `#F7FAFB` | Page canvas (deliberately never pure white) |
| `--ds-bg-warm` | `#F6F1E9` | Warm sand — alternate section fill |
| `--ds-ink` | `#17262B` | Body text |
| `--ds-font-display` | `'Space Grotesk'` | Headings, brand, buttons — **see OPEN-ITEMS #1** |
| `--ds-font-body` | `'Inter'` | Body copy |
| `--ds-font-mono` | `'Space Mono'` | Eyebrows — **see OPEN-ITEMS #1** |

## Page map (43)

**Home** — `index.html`

**Practice** (6) — `our-doctor.html` · `contact.html` · `insurance.html` · `reviews.html` ·
`patient-forms.html` · `book-appointment.html`

**Services hub** (1) — `services.html`

**Eye exams** (4) — `comprehensive-eye-exams.html` · `adult-eye-exams.html` ·
`senior-eye-exams.html` · `back-to-school-eye-exams.html`

**Children's vision** (4) — `childrens-eye-care.html` · `pediatric-eye-exams.html` ·
`childrens-contact-lenses.html` · `myopia-management.html`

**Medical eye care** (8) — `medical-eye-care.html` · `eye-health.html` · `dry-eye-treatment.html` ·
`glaucoma-management.html` · `diabetic-eye-exams.html` · `macular-degeneration.html` ·
`cataract-co-management.html` · `lasik-co-management.html`

**Urgent care** (5) — `emergency-eye-care.html` · `pink-eye-conjunctivitis.html` ·
`foreign-body-removal.html` · `red-eye-treatment.html` · `flashes-floaters.html`

**Contact lenses** (7) — `contact-lenses.html` · `contact-lens-exams.html` ·
`same-day-contacts.html` · `specialty-contacts.html` · `toric-contacts.html` ·
`gas-permeable-contacts.html` · `multifocal-contacts.html`

**Eyewear** (4) — `eyewear.html` · `designer-frames.html` · `sunglasses.html` · `kids-eyewear.html`

**Legal** (3) — `privacy-policy.html` · `disclaimer.html` · `accessibility.html`

## Editing

The build was designed to be edited by hand — findable containers, measurable boxes, safe
per-section asset swaps.

**Swapping an image:** count its references first (`grep -c 'old-image.jpg' *.html`). Several source
images are shared across dozens of pages, so overwriting a file in place silently changes every page
that uses it. The safe pattern is: add a **new** file and repoint only the container you mean to
change. Measure the target container's box before sizing the replacement.

For example, `assets/real/dr-hyder-portrait.png` was added rather than overwriting
`dr-hyder.jpg`, because that original is still referenced on 38 of the 43 pages.

## Deploying

See [`docs/DEPLOY.md`](docs/DEPLOY.md). Two things will bite you if you skip it: the empty
`.nojekyll` file is mandatory (without it GitHub Pages runs Jekyll and drops `_xorigin/`, so every
font 404s), and the Pages API reporting `status: "built"` is **not** proof your commit is live.

## Repository layout

```
.
├── index.html + 42 more pages   the site
├── chrome.css                   the only stylesheet
├── assets/                      images (real/ · gen/ · full-library/) + favicon.svg
├── _xorigin/                    self-hosted woff2 fonts
├── docs/                        provenance, deploy runbook, open items
├── audit.json                   clone-pipeline receipt
├── robots.txt                   Disallow: / — see PROVENANCE
├── 404.html
└── .nojekyll                    required; do not delete
```

## Licence and ownership

The page content, business name, practitioner name, address and phone number belong to Eye Trends
Vision & Glasses Center. This repository is a build artifact and carries no licence grant for that
content. **Verify rights before publishing.** See [`docs/PROVENANCE.md`](docs/PROVENANCE.md).
