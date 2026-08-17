# Open items

Known defects and outstanding work. Everything here was verified against the working tree on
**2026-08-17** with the command shown. Nothing in this list has been fixed — this build ships as-is,
and these are documented so nobody rediscovers them the hard way.

---

## 1. `Space Grotesk` and `Space Mono` never load — inherited from the source, not a clone defect

**Status:** verified. Affects all 43 pages. Present on the source site too.

The design tokens in `chrome.css` specify:

```css
--ds-font-display:  'Space Grotesk', system-ui, sans-serif;   /* headings, brand, buttons */
--ds-font-mono:     'Space Mono', ui-monospace, …;            /* eyebrows */
```

But **nothing anywhere loads either typeface.** `chrome.css` contains 56 `@font-face` blocks and all
56 are `'Inter'`. The only stylesheet any page links is `chrome.css` — there is no Google Fonts
stylesheet link, only a `preconnect` that fetches nothing.

So every heading, the brand **name** and every button falls through `--ds-font-display` to the
`system-ui` fallback. Eyebrows fall through a *different* chain: `--ds-font-mono` has no
`system-ui` in it, so they render in `ui-monospace` — along with the other rules consuming that
token, including `.brand-sub` (the "Vision & Glasses Center" line, `chrome.css` line 148), form
labels, hours rows, team roles and TOC titles. Two distinct wrong faces, not one — and note the
brand lockup is **split across both**: its first line takes the display chain, its second the mono
chain.

```bash
grep -A3 '@font-face' chrome.css | grep -o "font-family:[^;]*" | sort | uniq -c
#   56 font-family: 'Inter'
grep -ho 'rel="stylesheet"[^>]*' *.html | sort -u
#   rel="stylesheet" href="chrome.css"
```

**This is not something the clone broke.** The source site has the identical tokens and its Google
Fonts link requests only `Inter` and `Fraunces`:

```
https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&family=Fraunces:opsz,wght@9..144,400..700&display=swap
```

`Space Grotesk` is the `--ds-font-display` token value in the source's own `tokens.css`, and is
never loaded there either (`structural.css` mentions it once, inside a comment). The build is faithful to its source — which is precisely why the
round-trip pixel diff scored ~100%: both render in the same fallback.

**To fix,** pick one: self-host Space Grotesk + Space Mono into `_xorigin/` and add `@font-face`
blocks to `chrome.css` (matches the existing self-hosted pattern, no third-party request), or change
the tokens to a pairing that is actually loaded. Either way it is a **visible design change** — the
site will not look like it does now. Get sign-off before doing it.

---

## 2. 39 of 43 pages still use the original low-resolution imagery

**Status:** verified. Cosmetic, but the most visible quality problem in the build.

Only `index.html` received owner-supplied images. Three legal pages — `accessibility.html`,
`disclaimer.html` and `privacy-policy.html` — carry no imagery at all (zero `<img>` elements; their
only asset reference is the favicon). On the remaining 39, source assets are displayed well above
their intrinsic size:

| File | Intrinsic | Weight | Referenced on |
|---|---|---|---|
| `assets/real/dr-hyder.jpg` | **200 × 319** | 10 KB | 38 of 43 pages |
| `assets/real/practice-office.jpg` | **300 × 249** | 14 KB | 25 of 43 pages |
| `assets/real/optical-1.jpg` | **300 × 187** | 13 KB | 23 pages |
| `assets/real/optical-2.jpg` | **300 × 161** | 12 KB | 8 pages |

### How to compute the upscale — get this right first

Every one of these images is `object-fit: cover`, so the scale actually applied is
**`max(boxWidth / naturalWidth, boxHeight / naturalHeight)`**, not box-width ÷ natural-width.
Whenever the box is proportionally *taller* than the source, the height drives the scale and a
width-only calculation understates it badly — by 3.2× in the worst case below. Two independent
audits caught a width-only figure in this document; do not reintroduce one.

Also: several images are `loading="lazy"`. Measure them without scrolling and `naturalWidth`
reads **0**, which silently poisons any ratio you compute. Force `loading="eager"`, scroll, and
`decode()` before reading.

### Which image is worst depends entirely on the viewport

There is no single worst offender. A full sweep of every `<img>` on all 44 pages at 7 viewports
found the title changing hands three times:

| Viewport | Worst image in the build | Where | Upscale |
|---|---|---|---|
| 375 px | `optical-1.jpg` | `contact-lens-exams.html` | 4.06× |
| **768 px** | **`optical-2.jpg`** | **`glaucoma-management.html`** | **5.07× ← worst measured anywhere** |
| 900 px | `dr-hyder.jpg` | `adult-eye-exams.html` | 4.14× |
| 1024 px | `dr-hyder.jpg` | `adult-eye-exams.html` | 4.72× |
| 1280–1920 px | `optical-1.jpg` | `contact-lens-exams.html` | 4.27× → 6.40× |

**Mode 1 — an uncapped full-bleed hero.** On `contact-lens-exams.html`, `optical-1.jpg` (300 × 187)
renders at a width that **equals the viewport exactly**: `.container--full` sets `max-width:none`
and nothing up the chain caps it. Its width therefore grows without bound.

| Viewport | Box | True upscale |
|---|---|---|
| 375 px | 375 × 758 | 4.06× (height-driven) |
| 768 px | 768 × 576 | 3.08× (height-driven) |
| 1440 px | 1440 × 770 | 4.80× (width-driven) |
| **1920 px** | **1920 × 770** | **6.40×** (width-driven) |

Note the curve is **U-shaped, not monotonic** — it bottoms out around 768–900 px and rises again
toward the phone, because the box gets very tall as it narrows. Only the **width** tracks the
viewport; the height is content-driven (`aspect-ratio: auto`, `max-height: none`) and followed no
formula across the widths measured. `w / 300` is the correct upscale only above roughly 1235 px,
where the box finally becomes wider than the source's 1.604:1 aspect.

**Mode 2 — a capped container that collapses.** `dr-hyder.jpg` (200 × 319) is worst at a *mid*
viewport, because at ≤1024 px `.bw-founder` collapses to one column (`chrome.css` line 1349, inside
the `@media (max-width:1024px)` block opening at 1346) and the portrait takes the full content
width: 2.8× at 1280 px on `book-appointment.html`, 4.1× at 900 px and 4.71× at 1024 px on
`our-doctor.html`. (942 px is the bordered `.bw-founder__portrait` case; the borderless
`.dc-frame--tall` placements reach the full 944 px box, 4.72×.)

**Mode 3 — a CSS parse bug, which is the actual worst case.** The 5.07× reading is not a layout
decision at all; it is fallout from the dropped `.dc-frame` rule described in **item 7 below**.

**So: sweep the ladder and take the max per image, using the cover formula.** Neither end is
reliably the bad one. A production pass needs higher-resolution replacements sitewide, and the
full-bleed hero needs one at least 1920 px wide.

Follow the swap discipline in the README: count references first, add a **new** file, repoint only
the container you mean to change. `dr-hyder.jpg` alone is on 38 pages — overwriting it in place
would silently alter almost the whole site.

---

## 3. ~702 KB of the payload (12%) is dead weight

**Status:** verified. Safe to remove, but removal was deliberately deferred.

### `assets/full-library/` — 10 files, 555,292 B, referenced by nothing

Not one page or stylesheet references a `full-library/` path, and every file in it is a
byte-identical duplicate of a file that *is* referenced from `assets/gen/` or `assets/real/`:

```bash
for f in $(ls assets/full-library); do
  echo "$f -> $(grep -l "full-library/$f" *.html chrome.css 2>/dev/null | wc -l) refs"
done          # every line reports 0
```

Confirmed by SHA-256 as duplicate pairs: `designer-frames.jpg`, `exam-room.jpg`,
`hero-boutique.jpg`, `kids-eyewear.jpg`, `sunglasses.jpg` (twins in `assets/gen/`);
`dr-hyder.jpg`, `optical-1.jpg`, `optical-2.jpg`, `practice-office.jpg` and `favicon.svg` (twins in
`assets/real/` and `assets/`). All 43 pages reference `assets/favicon.svg`, never the library copy.

### 3 Fraunces webfonts — 146,676 B, referenced by nothing

`_xorigin/fonts.gstatic.com/s/fraunces/v38/*.woff2` were mirrored because the source's Google Fonts
link requests Fraunces — but **no `font-family` declaration in this build or in any of the three
source stylesheets ever uses Fraunces.** The only two occurrences of the string in `chrome.css` are
inside provenance comments recording that Google Fonts URL.

If you delete either group, do it the safe way: prove zero references first, then **move** the files
to a `_superseded/` folder rather than deleting, and confirm the removal shows up as a deletion in
`git status`.

---

## 4. No canonical, OpenGraph, Twitter or JSON-LD metadata on any page

**Status:** verified. Blocks a real launch; harmless while this is a build artifact.

Each of the 43 content pages carries `<title>`, `<meta name="description">`, `<meta charset>` and `<meta name="viewport">` — and no other metadata
(`404.html` differs in both directions — no description, but it does carry a robots meta). No
`<link rel="canonical">`,
no `og:*`, no `twitter:*`, no structured data. Sharing any URL produces a bare unfurl, and a search
engine gets no signal about which copy of this content is authoritative.

Related and more urgent — see `PROVENANCE.md`: there is **no `<meta name="robots">` on any of the 43
content pages either.** `404.html` is the only file in the build that carries one
(`<meta name="robots" content="noindex, nofollow">`) — it is the pattern to copy. The root
`robots.txt` requests that crawlers stay out, but that is a request, not an
access control, and it does not deindex URLs that are already known. For a build that duplicates a
real operating business with its own live website, per-page `noindex, nofollow` plus a canonical
pointing at `eyetrendsclearlake.com` is the control that actually works. **Adding it edits all 43
pages, so it was left out pending a decision.**

A local LocalBusiness / Optometric JSON-LD block would also be the obvious win if this ever goes
live — but only after the practice has confirmed the underlying facts, which has not happened.

---

## 5. `audit.json` ships in the deployed site

**Status:** verified. Low risk, trivially fixable.

The 23,993-byte clone-pipeline receipt is served publicly alongside the site. It exposes internal
build paths and the full source-URL map. It is kept because the folder was published complete and
deliberately unpruned; delete it, or move it under `docs/`, whenever you like — nothing references it.

It also records `bundle_root` as `…/Code/clones/jc1147.github.io`, a stale path from before the
directory was renamed to `Eye Trends`. Do not trust that field.

---

## 6. The 12/12 gate score in `audit.json` is narrower than it sounds

**Status:** context, not a defect.

The clone tooling's Stage-5 acceptance gates report 12/12 passed, 0 hard issues, 0 soft issues.
Those gates **do not verify that relative local CSS and image references resolve on disk** — Gate 10
only audits cross-origin URLs. A completely unstyled page has scored 12/12 under them before.

Treat the round-trip pixel diff as the real fidelity evidence: 43/43 pages passing across 258/258
viewports, 36 pages at 100.000%, lowest single viewport 99.983% on `book-appointment.html`.

---

## 7. 🔴 A malformed CSS comment silently drops 5 rules from the stylesheet

**Status:** verified in the browser against the deployed site. **This is a live rendering defect,
not a documentation issue — and it is the most serious item on this list.** It has not been fixed,
because fixing it changes site output and needs sign-off.

### The bug

Five block comments in `chrome.css` contain the literal string `.et-*/` inside prose describing the
namespace convention:

```
   this template's own; any .et-*/.bw-*/.row-- selector here is a SCOPED override, never a
                              ^^
                              this closes the comment
```

The `*/` inside `.et-*/` **terminates the block comment early**. Everything after it — the rest of
the English prose, the real `*/`, and then the first real rule that follows — is fed to the CSS
parser as one malformed construct and discarded. The parser recovers at the *second* rule.

Affected comment lines: **1441, 1670, 1809, 1928, 2057**.

### What it costs — 5 rules never reach the page

Verified by reading the live CSSOM (`document.styleSheets`), not by inspection:

| Comment | Rule silently dropped | In CSSOM | Next rule down | In CSSOM |
|---|---|---|---|---|
| 1441 | `.ct-hero` | **0** | `.ct-hero::before` | 1 |
| 1670 | `.ia-hero` | **0** | `.ia-hero__media` | 1 |
| 1809 | `.od-hero__quote` | **0** | — | — |
| 1928 | `.thb-call` | **0** | `.thb-call svg` | 1 |
| 2057 | `.dc-frame` | **0** | `.dc-frame--tall` | 1 |

`.dc-frame` is used on **21 pages**. With its base rule missing, those media frames lose everything
it was supposed to give them:

| Property | Intended | Actually computed |
|---|---|---|
| `position` | `relative` | `static` |
| `aspect-ratio` | `4 / 3` | `16 / 11` (inherited from elsewhere) |
| `max-height` | `clamp(320px, 40vw, 452px)` | `400px` |
| `overflow` | `hidden` | `visible` |
| `border-radius` | `var(--ds-r-surface)` | `0px` |

Losing `position:relative` is what causes the **worst image upscale in the build**: the
absolutely-positioned `.dc-frame__img` child no longer has `.dc-frame` as its containing block, so
it escapes to a further-out ancestor and stretches to 707 × 816 — blowing a 300 × 161 source up to
**5.07×** on `glaucoma-management.html` at 768 px (see item 2).

### Reproduce it

```bash
grep -n '\.et-\*/' chrome.css        # the five comment lines
```
```js
// in the browser console on any page
const s = [...document.styleSheets].find(s => (s.href||'').includes('chrome.css'));
[...s.cssRules].filter(r => r.selectorText === '.dc-frame').length   // → 0
[...s.cssRules].filter(r => r.selectorText === '.dc-frame--tall').length // → 1
```

### The fix (not applied)

One character per site: change `.et-*/` to `.et-` (or `.et-\*/`, or reword) in each of the five
comments, then re-verify that all five rules appear in the CSSOM. It is a low-risk edit to comment
text only, but it **will change the rendered output** on the affected pages — restoring rounded
corners, overflow clipping and the intended aspect ratios — so it is a visible design change and
needs approval before shipping.

**Provenance note:** this is almost certainly inherited from the source build rather than introduced
by the clone, since `chrome.css` was consolidated verbatim from the source's stylesheets. Worth
telling whoever maintains the original.
