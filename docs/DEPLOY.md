# Deploy runbook

The site is static with no build step, so deploying is "put the files on a web server." The traps
below are the ones this specific project has already hit — both cost real debugging time.

**Current deployment:** GitHub Pages, `carlcelinodspnza/eye-trends`, branch `main`, path `/` →
https://carlcelinodspnza.github.io/eye-trends/

## Trap 1 — `.nojekyll` is mandatory

GitHub Pages runs Jekyll by default, and **Jekyll excludes any file or directory whose name starts
with an underscore.** This site keeps all 10 self-hosted fonts in `_xorigin/`. Without the empty
`.nojekyll` file at the repo root, every font 404s on the live site.

The failure is invisible locally — it only appears on the deployed URL. **Do not delete
`.nojekyll`.**

## Trap 2 — `status: "built"` does not mean your commit is live

The Pages API will happily report `status: "built"` while still serving the **previous** commit.
This has produced false "it's deployed" conclusions on this exact repo, twice. Never trust the
status field alone — assert the SHA.

```bash
REPO=carlcelinodspnza/eye-trends
PUSHED=$(git rev-parse HEAD)
BUILT=$(gh api repos/$REPO/pages/builds/latest --jq '.commit')

# assert, don't eyeball — and force a rebuild if Pages is lagging
if [ "$PUSHED" = "$BUILT" ]; then
  echo "live: $BUILT"
else
  echo "LAGGING: pages=$BUILT pushed=$PUSHED"
  gh api -X POST repos/$REPO/pages/builds
fi
```

Belt and braces: after the SHA matches, fetch a real asset and grep it for a string you know is new
in this build. A served byte is the only proof that counts.

## Routine deploy

```bash
cd "Eye Trends"
git add -A
git commit -m "…"
git push                       # origin -> carlcelinodspnza/eye-trends
```

Pages rebuilds in roughly 15–60 seconds.

## Post-deploy verification

Run every check against the **live URL**, never the local copy.

```bash
BASE=https://carlcelinodspnza.github.io/eye-trends

# 1. pages respond
for p in "" services.html book-appointment.html contact.html toric-contacts.html; do
  printf '%-28s %s\n' "$p" "$(curl -s -o /dev/null -w '%{http_code}' "$BASE/$p")"
done

# 2. stylesheet is complete, not truncated
curl -s -o /dev/null -w 'chrome.css %{http_code} %{size_download}\n' "$BASE/chrome.css"
#    expect: 200 223507

# 3. THE font check — a 404 here means Jekyll ate _xorigin/
curl -s -o /dev/null -w 'font %{http_code}\n' \
  "$BASE/_xorigin/fonts.gstatic.com/s/inter/v20/UcC73FwrK3iLTeHuS_nVMrMxCp50SjIa0ZL7W0Q5n-wU.woff2"

# 4. images
for a in assets/favicon.svg assets/real/dr-hyder-portrait.png assets/gen/exam-room.jpg; do
  printf '%-40s %s\n' "$a" "$(curl -s -o /dev/null -w '%{http_code}' "$BASE/$a")"
done

# 5. byte-for-byte proof the deploy serves what you built
shasum -a 256 index.html | cut -d' ' -f1
curl -s "$BASE/" | shasum -a 256 | cut -d' ' -f1
#    these two must match exactly
```

Anything other than `200` on checks 1–4, or a hash mismatch on 5, means the deploy is **not** done.

## Moving to a different host or subpath

No URL rewriting is needed. Every reference in the build is relative — 0 root-absolute `href="/…"`
or `src="/…"`, and `chrome.css` reaches the fonts via relative `url(_xorigin/…)`. The site serves
correctly from a domain root, from a project subpath, or from any nested folder.

Two things to handle if it moves to a real domain:

1. Add a `CNAME` file containing the domain, and point DNS at GitHub Pages.
2. Revisit the search-engine posture. `robots.txt` currently disallows everything — appropriate
   while this is a build artifact, wrong for a live site. See `PROVENANCE.md` before removing it,
   because the reason it is there is that this build duplicates a real business that has its own
   website.

## Enabling Pages on a fresh copy of this repo

```bash
gh api -X POST repos/<owner>/<repo>/pages \
  -f 'source[branch]=main' -f 'source[path]=/'
gh api repos/<owner>/<repo>/pages --jq '{status, html_url, source}'
```

Then run the SHA assertion from Trap 2 and the verification block above.
