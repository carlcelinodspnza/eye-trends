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

So every heading, the brand lockup and every button falls through `--ds-font-display` to the
`system-ui` fallback. Eyebrows fall through a *different* chain: `--ds-font-mono` has no
`system-ui` in it, so they — along with the other rules consuming that token (form labels, hours
rows, team roles, TOC titles) — render in `ui-monospace`. Two distinct wrong faces, not one.

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

`Space Grotesk` appears in the source's `tokens.css` and `structural.css` as a token value and is
never loaded there either. The build is faithful to its source — which is precisely why the
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

Measured on the deployed site rather than estimated: `dr-hyder.jpg` renders at **563 × 440** on
`book-appointment.html` at a 1280 px viewport, from a 200 × 319 source — a **2.8× upscale**, and
that is the widest it gets anywhere in the build. Noticeable, not catastrophic. A production pass
still wants higher-resolution replacements sitewide.

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

Every page has `<title>` and `<meta name="description">` and nothing else. No `<link rel="canonical">`,
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
