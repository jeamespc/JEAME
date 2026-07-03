# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-file HTML/CSS/JS portfolio site for **Cem Izgi** (studio name **JEAME**), an interior/product/spatial designer. The entire site is `index.html` at the repo root — one document containing all markup, a `<style>` block, and several `<script>` blocks. No build step, no framework, no bundler, no package.json, no dependencies. Edits are direct string replacements in one large HTML file, not component files.

Live site: **jeame.space** (see `CNAME`). This repo (`jeamespc/JEAME` on GitHub) deploys straight to GitHub Pages from `main` — pushing to `main` is equivalent to publishing.

## Repo contents

- `index.html` — the entire site.
- `CNAME` — custom domain (`jeame.space`) for GitHub Pages.
- `renders/` — the live image folder the site's JS probes for (see "Image loading convention" below). Keep filenames matching the conventions exactly, or an image will just silently fail to appear.
- `smalliconjeame.png` — favicon/brand icon.

The original design/working files (PSDs, brand/portfolio/resume reference PDFs, high-resolution source renders) live in the author's local working folder outside this repo and aren't tracked here — only the final `index.html` and web-sized `renders/` get pushed.

## Commands

There is no build/lint/test tooling. Development loop is:
- Edit `index.html` directly.
- Open it in a browser to test (double-click / `start index.html`) — no dev server required, though one can be used for convenience.
- Test in **both** a mouse+keyboard context and a touch/mobile emulation context — a large part of the interaction model branches on `isTouchDevice`/`(hover:none)`/`(pointer:fine)` and the two modes behave differently (see Mobile section below).
- No CI: pushing to `main` deploys directly. There's no staging environment, so verify changes locally in a browser before pushing.

## The person and the brand

- **Cem Izgi** — interior, product, and urban/spatial designer. Studio/brand name: **JEAME**.
- Footer brand statement: *"Spaces designed to provoke emotion, challenge habit, and elevate the human interaction."* Design philosophy leans on cinematic/theatrical scenography and instinct-based sketching before refinement.
- Contact info appears in **three separately hand-written places** (not templated): the main footer `.contact-grid`, and both `.po-contact-grid` blocks inside the two project-overlay footer templates. Any contact-info change must be made in all three or they drift out of sync.
  - Email: `jeame.spc@gmail.com`
  - Instagram: `@jeame.spc` (brand) and `@cemizgi04` (personal) — both listed, in that order.
  - Phone (DE): `+49 173 4152475`; Phone (IT): `+39 351 6581099`

## Design system

CSS custom properties on `:root`:
```css
--red:#AD3745;   --blue:#283583;   --orange:#F59D23; --green:#14672F;
--black:#0A0A0A; --paper:#FAFAF7;  --line:#0A0A0A;   --mono:#1C1C1C;
```
- **Typography:** Arial Black/Arial (weight 900) everywhere except the Tales book-reading view, which uses Georgia/Times New Roman to feel like a printed page.
- **Category color coding:** Interior → blue, Product → red, Urban and Spatial → green, Stories section → orange, Sketchbook → red, Tales → blue, Photography → black. Graphics/3D (unbuilt placeholders) → orange/green respectively.
- **Layout spine:** most section content aligns to a left margin of `5.4vw` desktop / `4vw` mobile. Exception: the Tales page title/hint deliberately use `8vw` to match the topbar `JEAME` logo — don't "fix" this back to the standard spine value.
- **Custom cursor:** native cursor is hidden on `(pointer:fine)` and replaced by `#jeame-cursor` following the pointer via JS. The hiding rule is `*{ cursor:none !important; }` inside `@media (pointer:fine)` — the `!important` and universal selector are load-bearing: plenty of interactive elements (`.hero-draw-btn`, `.po-gallery-tile.has-img`, etc.) set their own `cursor:pointer` with higher selector specificity further down the stylesheet, which used to win over a plain `a, button, .shape, [tabindex]{cursor:none}` rule and let the native pointer/hand cursor bleed through on hover. If you add a new interactive element with its own `cursor:` rule, this `!important` override still applies — you don't need to special-case it.
  - `.on-dark` class switches the cursor to a white difference-blend dot over dark backgrounds, controlled by `DARK_ZONE_SELECTOR` (currently `'#about, .po-footer, .gallery-lightbox, #photography-page'`). **Any new full-page overlay with a dark background must be added to this selector**, or the cursor renders invisible on it.
  - Touch/coarse-pointer devices get the real system cursor and their own tap-driven interaction patterns instead of hover.
- **Hero shape hit-testing:** the hero nav shapes are SVG paths; a plain filled `<rect>`/`<circle>` is hit-testable everywhere inside its bounds, but a compound path with a knockout hole (e.g. the Interior blue square with the circular cutout) only registers hits on the actually-painted ring by default (`pointer-events:visiblePainted`, hit-tested against the fill-rule geometry) — the hollow middle is a dead zone. The fix already applied for Interior is an invisible sibling `<rect fill="transparent" pointer-events="all">` spanning the shape's full bounding box, placed inside the same `<g class="shape">`. Any future shape with a hole needs the same treatment.

## Architecture

Everything is one document, sections shown/hidden via CSS classes driven by `location.hash` (`hashchange` listeners — no router library). File organization, top to bottom:
1. `<head>` — GA (`gtag.js`, ID `G-JB929WWNZP`) and the root `<style>` block (CSS vars → cursor/hero shape CSS → per-feature CSS for Sketchbook/Tales/Photography/gallery lightbox/project overlay/CV overlay).
2. HTML body, in DOM order: Hero (`#top`) → category sections `#interior`/`#product`/`#urban` → Stories section `#stories` (links out, not projects) → footer/about (`#about`) → project detail overlay (`#project-overlay`) → CV overlay (`#cv-overlay`) → Sketchbook/Tales/Photography full-page overlays → shared gallery lightbox (`#gallery-lightbox`).
3. `<script>` blocks, roughly: shared utilities + `PROJECTS` data → project overlay logic → CV overlay logic → hero nav/cursor logic → Sketchbook logic → Tales logic → Photography logic → hero drawing logic → gallery lightbox logic.

Use `grep -n` for the `<!-- ===...=== -->` HTML banners and `// ---------- NAME ----------` JS banners to navigate — they mark each feature zone.

### Hash routing pattern

Every full-page overlay (project detail, CV, Sketchbook, Tales, Photography) follows the same hand-written sync-function pattern: on `hashchange` (and once on load), if the hash matches, add `.show` + `aria-hidden="false"` + a body scroll-lock class + `scrollTo(0,0)`; else remove all three. **Any new overlay must pair the scroll-lock class `add` with a `remove` in the same function's else-branch** — omitting the `remove` was a real, previously-fixed bug (Tales/Photography leaked `tl-open`/`ph-open`, which tied in CSS specificity with the permanent `body.unlocked` rule and — because it appeared later in stylesheet order — won, permanently breaking page scroll after the overlay was opened once). Sketchbook (`sb-open`) always did this correctly; use it as the reference implementation.

Recognized hashes: `''`, `#interior`, `#product`, `#urban`, `#stories`, `#{project-key}` (any `PROJECTS` key), `#cv`, `#sketchbook`, `#tales`, `#photography`. `syncProjectOverlay()` no-ops safely on unrecognized hashes.

### Project data model

Projects live in one JS object, `PROJECTS`, keyed by a URL-safe slug that doubles as the hash:
```js
const PROJECTS = {
  'the-craft': {
    title, year, time,       // time format 'MON+YYYY' or a range like 'DEC2025-MAR2026' — drives sort order
    type, role,
    category,                 // 'interior' | 'product' | 'urban'
    accent, dotColor,         // hex colors for gallery placeholders / spine dot
    art: `<svg>...</svg>`,    // inline icon for the overlay header
    paragraphs: [ ... ]        // rendered as <p> tags
  },
  // ...
};
```
`CATEGORY_META` maps `category` → `{hash, label}` for back-links/headers. `PROJECT_ORDER` and `parseEndDateStr` derive sort order and "Next project" links automatically from each project's `time` field — never hand-maintain an ordering array.

**To add a project:** add a `PROJECTS` entry AND a matching `<a class="row" href="#your-key">` inside the right `section.cat` — these are hand-written and must be kept in sync manually; nothing generates the rows from `PROJECTS`.

### Image loading convention (no upload/manifest system)

Images are referenced by predictable paths under `renders/`, managed outside the codebase. JS **probes** for files (`probeImage()`/`probeVideo()`, tries `Image()`/loads and resolves true/false) and silently stops on a miss — there is no manifest, so a missing or misnamed file just fails to appear with no error. If "an image isn't showing up," check the filename/number sequence first, not the code.

| What | Path pattern |
|---|---|
| Project gallery images | `renders/{project-key}-{n}.{ext}`, sequential from 1, no gaps |
| Project cover (optional) | `renders/{project-key}-cover.{ext}` |
| Stories row thumbnails | `renders/{tales,sketchbook,graphics,3d,photography}.png` |
| Sketchbook sketches | `renders/sketchbook-{n}.{ext}`, sequential, cap 24 |
| Photography filmstrips | `renders/photography-{1,2}-{n}.{ext}`, sequential, reuses `loadGallery()` |

Accepted extensions: `.jpg`/`.jpeg`/`.png` everywhere (`GALLERY_EXTENSIONS`); some gallery slots also accept `.mp4` (`GALLERY_VIDEO_EXTENSIONS`).

**Resolution is cached and preloaded, not just probed on open.** `getCoverSrc(key)` / `getGalleryItems(key)` memoize their resolved results per project key in module-level `Map`s (`coverSrcCache`, `galleryItemsCache`), and `resolveGalleryItem()` probes all accepted extensions for a slot **in parallel** (`Promise.all`) rather than one at a time. `loadGallery()`/`loadCover()` (used when a project overlay actually opens) just await these cached promises. Separately, delegated `pointerover`/`focusin` listeners on `document` call `preloadProject(key)` for whatever project row is hovered/focused, debounced ~120ms (`schedulePreload`) so a quick mouse sweep or held-Tab pass over several rows doesn't fire a full probe burst (up to ~70 speculative requests) for every row it crosses. Net effect: by the time someone actually opens a project, its images are usually already downloading or cached. If you touch this, keep the cache keyed by project slug and keep probing parallel — the old sequential-await version was the reason opening a project used to visibly stall on a blank gallery.

### Shared systems (reuse these, don't reimplement)

- **`loadGallery()` / `renderGalleryTiles()` / `probeImage()` / `probeVideo()`** — the horizontally-scrollable filmstrip gallery, used by project pages and reused as-is by Photography's two reels.
- **`openLightboxFromGallery(galleryEl, startIndex)`** (`window.openLightboxFromGallery`) — shared full-screen lightbox; reads `.has-img` tiles from whatever gallery element is passed, so it works for any gallery with no extra config. Keyboard arrows, backdrop click to close, touch swipe (40px threshold, horizontal-only).
- **`.cat-preview` + `showPreviewFor()`/`hidePreview()`** — project-row hover/tap preview. Behavior forks on `isTouchDevice`:
  - **Non-touch (`body.cursor-preview-mode`, and only above `min-width:901px`):** the preview is `position:fixed` and follows the cursor with an eased `requestAnimationFrame` loop (`followLoop()`/`setPreviewTarget()`), offset to sit just right of and vertically centered on the pointer, clamped to the viewport. `setPreviewTarget(x, y, snap)` snaps instantly (no lerp) the moment a *different* row is entered, so the image doesn't visibly fly in from its last position.
  - **Touch, or narrower than 901px:** falls back to the original static panel positioned near the section `<h2>` (`@media (max-width:900px)` / default `.cat-preview` rules), shown via tap-to-preview (first tap previews, second tap on the same row navigates) with a `scrollIntoView({block:'center'})` call so it isn't left off-screen on long lists.
  
  If you rework this, preserve both branches — they're gated by the `isTouchDevice` check computed once near the top of this script, not by a CSS media query alone.
- **`markProjectVisited(key)`** — session-only (`sessionStorage`) visited-row indicator (`.row-visited`), resets each fresh visit.

### The three Stories sub-pages are intentionally NOT a shared template

Each has its own distinct visual language by design:
- **Sketchbook** (`#sketchbook-page`) — dark canvas, cursor-following spotlight (`.sb-spotlight`), scattered/rotated sketch images, click-to-focus zoom, no drop-shadow (deliberate). Scroll-lock: `sb-open`.
- **Tales** (`#tales-page`) — bookshelf of spines (`TALES_BOOKS`), opens a reader (`#tlReader`). Book type `'story'` (book-1 only) is hand-typed in `BOOK_1_STORY` as `{type:'h'|'p', text}` blocks, laid out by a custom `paginateBlocks()` engine that measures an offscreen clone and fills real book-style pages (re-paginates on resize). Type `'pages'` is legacy/unused (scanned-image pagination, still functional if needed). Type `'text'` is a simple placeholder card view (books 2–4). Page turning works three ways simultaneously (arrow buttons, keyboard arrows, click-page-half) — preserve all three if touched. Title/hint use `8vw` left offset (see Design system). Scroll-lock: `tl-open`.
- **Photography** (`#photography-page`) — two dark titled filmstrips (`REELS` array) with sprocket-hole border styling, directly reusing the gallery/lightbox system with two independent containers (`#phStrip1`/`#phStrip2`). Scroll-lock: `ph-open`.

### Hero drawing feature (private, client-only)

The hero (`#heroCanvas`, toggled via `.hero.drawing`) lets visitors doodle over the logo with pen/eraser. Strokes save only to that browser's own `localStorage` (`jeame-hero-drawing-v1`) — nothing is ever sent anywhere, no cross-device/visitor sharing. Keep this purely client-side if modified.

## Mobile-specific behavior

`isTouchDevice` (`matchMedia('(hover:none)')`, `ontouchstart`, `maxTouchPoints`) gates two-tap-to-navigate patterns: hero shape nav (first tap arms, second navigates) and project row previews (first tap shows preview, second tap on the same row navigates). Don't bind hover handlers unconditionally — synthetic mouseover/mouseout fires after real taps on touchscreens and can cause flicker if not guarded by `isTouchDevice`.

## Known placeholder/incomplete content (not bugs)

- **Graphics** and **3D** Stories rows: static, non-clickable, "Coming soon" — not built yet, on purpose.
- **Tales books 2–4** ("A Booklet", "A Report", "Field Notes"): placeholder titles/blurbs.
- **Tales book-1 manuscript** (`BOOK_1_STORY`): placeholder text; the real *"Tales from a Young Man's Old Life"* manuscript hasn't been supplied yet.
- **Photography strip titles/image keys** ("Strip One"/"Strip Two", `photography-1`/`photography-2`): placeholders, no real photos supplied yet.

These degrade gracefully by design per the image-probing convention above — don't "fix" them without new content from Cem.
