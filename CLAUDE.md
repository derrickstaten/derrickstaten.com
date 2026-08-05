# CLAUDE.md

This file provides guidance to Claude Code when working in this repository.

## What this is

A hand-written static site for derrickstaten.com. No build step, no
`package.json`, no framework — plain HTML plus one shared stylesheet, served
straight off GitHub Pages (`CNAME`, `.nojekyll`). Photos live in `images/`.

| File | What it is |
| --- | --- |
| `index.html` | **Temporary holding page** (see below). Not the real homepage. |
| `_index.html` | The real homepage: lead-gen page — hero, Track record, two offerings, contact form. |
| `_my-story.html` | The timeline: hero, then "chapter" blocks (year + location tags + text card + a polaroid-style photo), newest at the top, oldest at the bottom. |
| `thanks.html` | Form receipt, `noindex`. Formspree redirects here. |
| `styles.css` | Every page's styles. Linked with a `?v=N` cache-buster — **bump N whenever you change the CSS**, or Derrick will test a stale file. |
| `robots.txt` | Open to all crawlers (`Allow: /`) except `/kamikaze/`, which every listed group explicitly `Disallow`s — see below. |
| `kamikaze/index.html` | A JS port of a QBasic game Derrick wrote in high school (2004), a Claude-built line-for-line transcription. Standalone, self-contained, full-viewport (`100dvh`, `body{overflow:hidden}`, a global keydown listener). Deliberately unlinked from the site nav — findable only via the console easter egg below. `noindex, nofollow` in its own `<meta>`. **Don't refactor, reformat, or "improve" the QBasic shim / game logic in this file — it's verified line-for-line against the original source.** Only layout/integration changes are fair game. See Ahead DS-17 for the full history. |

There are now a few small scripts on the site, all intentionally minimal:
- ~10 lines at the bottom of `_my-story.html` so the Escape key closes the
  screenshot lightbox. Everything else there is CSS, including the lightbox
  itself (`:target`).
- A matching ~5-line console easter egg at the bottom of `index.html`,
  `_index.html`, and `_my-story.html`: logs a console message and exposes
  `window.kamikaze()`, which redirects to `/kamikaze/`. Keep these three
  copies in sync if the message or target path changes.

Don't reach for JS without a reason — but "no JS at all" is no longer true,
so don't state it.

### The site is currently hidden

The real pages are renamed behind a leading underscore so nothing at a
guessable URL serves the in-progress site; underscore-prefixed files are served
normally because `.nojekyll` is present. To go live:

1. **Delete the `<meta name="robots" content="noindex, follow">` tag from
   `_index.html` and `_my-story.html`.** Do this first and don't forget it —
   leaving it in place makes the launched site invisible to search engines.
   Each one has a loud comment above it.
2. Delete (or rename) `index.html`, the holding page.
3. `git mv _index.html index.html` and `git mv _my-story.html my-story.html`.
4. Rewrite the `_index.html` / `_my-story.html` hrefs across all pages back to
   `index.html` / `my-story.html`.
5. Restore `og:url` on both pages to the public URLs.

`robots.txt` allows everything, including AI/LLM crawlers, and needs no change
at launch. Crawlers may fetch the underscore pages today; the `noindex` above
only keeps those temporary URLs out of *search indexes*, so they can't be
indexed and then 404 when the files are renamed.

Full context lives in Ahead: **DS-32** (the critique driving the rework) and
**DS-38** (what was changed and why) — both closed; remaining work is tracked
under **DS-2**.

The **testimonials section** in `_index.html` is live with three real quotes —
Mike Molinet, Alex Bauer, Tomas Vacek, in that order (**DS-34**, closed). Mike's
and Alex's are from 2026, Tomas's is a 2014 LinkedIn recommendation; the dates
are intentionally not shown. No placeholder cards remain; the `.is-placeholder`
CSS is kept only in case a fourth quote (Amanda Bradford, **DS-54**) lands.
Don't ship placeholder quotes.

## Workflow: no worktree, no commit-per-change

Derrick works directly against `main` here — there's no branch/PR flow for this
repo. `.claude/settings.json` sets `worktree.bgIsolation: "none"` so background
sessions don't need `EnterWorktree` before editing; edit the HTML, `styles.css`,
and `images/` directly.

- Don't create a git commit unless explicitly asked.
- Don't spin up a worktree for routine edits (copy swaps, text tweaks, CSS
  tweaks). This is a single always-current working tree, not a per-issue flow.

## Committing

- **Use the GitHub noreply alias for every commit, not Derrick's real email.**
  Find it from existing history (`git log --format='%an <%ae>'`) rather than
  hardcoding it here, in case it ever changes — as of this writing it's
  `61054169+derrickstaten@users.noreply.github.com`. Pass it per-commit via
  `-c` flags so it never touches global or repo git config:
  ```bash
  git -c user.name="Derrick Staten" -c user.email="<noreply alias>" commit -m "..."
  ```
- **Don't push until Derrick has tested the change**, even after committing —
  he often wants to verify locally (see "Verifying a change actually
  rendered" below) before it goes live. Exception: when he explicitly says to
  deploy/ship/push now, do it immediately without waiting.

## Working with photos

Derrick drops real photos (often `.HEIC` from an iPhone) straight into
`images/` or the repo root. When asked to wire one in:

1. Convert with `sips -s format jpeg -Z 2400 in.HEIC --out images/name.jpg`
   (2400px longest side keeps files reasonably small for a static site).
2. **Actually view the converted JPEG with the Read tool before wiring it into
   the page.** HEIC files from iPhone frequently have no EXIF orientation tag
   at all (`sips -g orientation` reports `<nil>`), so `sips` can't auto-rotate
   them — whatever the sensor wrote is what you get. Check it looks right.
   - That said: Derrick has said before that photos should NOT be rotated
     from whatever the plain, unrotated `sips` conversion produces — even when
     a rotation looks visually "correct" to me. If a photo looks sideways,
     ask before manually rotating it; don't silently "fix" it.
3. Pick the right slot:
   - **`.chapter-bg`** — the full-bleed background photo behind a chapter or a
     `.section-photo`. Wide/epic shots work best.
   - **`.chapter-polaroid`** — a square-cropped personal print, rotated
     slightly, one per chapter. `object-fit: cover` crops to a square; for
     content that shouldn't be cropped (app screenshots, mockups) add
     `class="fit-contain"` to the `<img>` to letterbox it on black instead.
   - **`.chapter-shots`** — the Sightline chapter only: a loose overlapping
     pile of three wide product screenshots in the polaroid's grid slot, each
     opening a lightbox. Rotation and offsets are per-child in the CSS.

Rotation on prints uses the **`rotate` property, never `transform`** — the
parallax animation owns `transform`, and setting it would silently kill the
drift.

### Repositioning a background image ("shift the focus higher/lower")

`object-position` on the `<img>` inside `.chapter-bg` controls framing, via a
`--bg-pos` custom property set inline on the div:

```html
<div class="chapter-bg" style="--bg: url('images/x.jpg'); --bg-pos: center 20%;">
```

There used to be a second rendering path — a `background-image` +
`background-attachment: fixed` fallback for browsers without scroll-driven
animation — and older guidance said to fix both. **That fallback is gone**;
`background-attachment: fixed` forced a repaint on every scroll tick in desktop
Safari. Browsers without `animation-timeline: view()` now just get the same
static image, so there is only one path to fix.

Note that `--bg` itself is vestigial: it's still set inline on every
`.chapter-bg`, but nothing in the CSS reads it any more. Harmless, but don't
assume changing it does anything.

**Before reaching for `object-position` at all**, check the image's aspect ratio
against the crop box, which is roughly `container-aspect / 1.26`. If the source
photo is already close to that, there's almost no vertical overflow to
redistribute and the change will be imperceptible — this bit us once already.
In that case **bake the crop into the file** (e.g. Pillow: crop off the bottom
N% before saving). It works identically everywhere and doesn't depend on the
viewer's viewport.

## Verifying a change actually rendered

No server needed — open `file:///…/_index.html` directly. To check how something
actually renders (rather than guessing from the source), drive a real headless
browser:

```bash
npm install playwright --no-save   # in a scratch dir
node -e "..."
```

- Screenshot **the specific element** (`.chapter`, `.proof-grid`, `.site-nav`),
  not the whole page — very tall Chrome headless screenshots have produced
  corrupted/blurry output here before.
- Images below the fold are `loading="lazy"`, so a naive "any broken images?"
  check will report false positives. Scroll the page first, then assert.
- Worth asserting alongside a visual check: no horizontal overflow
  (`scrollWidth > innerWidth`), no failed requests, no console errors.
