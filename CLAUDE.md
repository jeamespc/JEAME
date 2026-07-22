# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-file HTML/CSS/JS portfolio site for **Cem Izgi** (studio name **JEAME**), an interior/product/spatial designer. The entire site is `index.html` at the repo root — one document containing all markup, a `<style>` block, and several `<script>` blocks. No build step, no framework, no bundler, no package.json, no dependencies. Edits are direct string replacements in one large HTML file, not component files.

Live site: **jeame.space** (see `clone/JEAME/CNAME`). Deployment is via GitHub Pages from the `jeamespc/JEAME` repo (see "Deployment / folder layout" below) — this working directory is not itself a git repo.

## Commands

There is no build/lint/test tooling. Development loop is:
- Edit `index.html` directly.
- Open it in a browser to test (double-click / `start index.html` on Windows) — no dev server required, though one can be used for convenience.
- Test in **both** a mouse+keyboard context and a touch/mobile emulation context — a large part of the interaction model branches on `isTouchDevice`/`(hover:none)`/`(pointer:fine)` and the two modes behave differently (see Mobile section below).

## Deployment / folder layout

This directory (`SITE FOLDER`) is the **working copy** where the live `index.html` is authored, plus asset staging areas:
- `index.html` — the actual working file, edited directly.
- `renders/` — the *live* image folder, matching the naming convention the site's JS expects (this is what gets published).
- `big renders/renders/` — higher-resolution/original source images, not what the site loads directly.
- `raw files/` — source design files (e.g. `.psd`).
- `reference files/` — brand/portfolio/resume PDFs (`CemIzgi_PersonalBrand.pdf`, `Cem_Izgi_PortfolioX.pdf`, `CemIzgi_Resume.pdf`) used for tone/content reference when writing copy.
- `clone/JEAME/` — a separate git checkout of the deployed GitHub Pages repo (`https://github.com/jeamespc/JEAME.git`, custom domain `jeame.space` via `CNAME`). This is the publish target: the finished `index.html` (and matching `renders/`) get copied/committed there to go live. Don't assume it's in sync with the working copy here — check before treating it as source of truth.

## The person and the brand

- **Cem Izgi** — interior, product, and urban/spatial designer. Studio/brand name: **JEAME**.
- Footer brand statement: *"Spaces designed to provoke emotion, challenge habit, and elevate the human interaction."* Design philosophy leans on cinematic/theatrical scenography and instinct-based sketching before refinement.
- Contact info appears in **three separately hand-written places** (not templated): the main footer `.contact-grid`, and both `.po-contact-grid` blocks inside the two project-overlay footer templates. Any contact-info change must be made in all three or they drift out of sync.
  - Email: `jeame.spc@gmail.com`
  - Instagram: `@jeame.spc` (brand) and `@cemizgi04` (personal) — both listed, in that order.
  - Phone (DE): `+49 173 4152475`; Phone (IT): `+39 351 6581099`
- For real project content, tone, or positioning changes, consult the PDFs in `reference files/` rather than inventing content.

## Design system

CSS custom properties on `:root`:
```css
--red:#AD3745;   --blue:#283583;   --orange:#F59D23; --green:#14672F;
--black:#0A0A0A; --paper:#FAFAF7;  --line:#0A0A0A;   --mono:#1C1C1C;
```
- **Typography:** Arial Black/Arial (weight 900) everywhere except the Archives reader, which uses Courier New/monospace for the typed body text (typewriter effect) and Georgia/Times New Roman italic just for the small group-description subtitles.
- **Category color coding:** Interior → blue, Product → red, Urban and Spatial → green, Stories section → orange, Sketchbook → red, Archives → blue, Photography → black, Graphics → orange, 3D → green.
- **Layout spine:** most section content aligns to a left margin of `5.4vw` desktop / `4vw` mobile. Exception: the Archives page title/hint deliberately use `8vw` to match the topbar `JEAME` logo — don't "fix" this back to the standard spine value.
- **Custom cursor:** hidden native cursor on `(pointer:fine)`, replaced by `#jeame-cursor` following the pointer via JS.
  - No more static dark-zone selector/class. On every animation frame, JS walks up from `elementFromPoint(x,y)` until it finds an element with a non-transparent `background-color`, computes its luminance, and sets the dot to white or `var(--black)` directly — whichever contrasts. A new overlay with a dark background just works automatically; nothing needs to be registered for it. (An earlier version used a `.on-dark` class driven by a `DARK_ZONE_SELECTOR` allowlist; that was replaced because `mix-blend-mode` compositing against the page's own paint didn't render reliably.)
  - One deliberate exception: hovering a `.shape` (hero shapes) camouflages the cursor into that shape's own `--shape-color` instead of contrast-picking (`blend-shape` class).
  - Touch/coarse-pointer devices get the real system cursor and their own tap-driven interaction patterns instead of hover.

## Architecture

Everything is one document, sections shown/hidden via CSS classes driven by `location.hash` (`hashchange` listeners — no router library). File organization, top to bottom:
1. `<head>` — GA (`gtag.js`, ID `G-JB929WWNZP`) and the root `<style>` block (CSS vars → cursor/hero shape CSS → per-feature CSS for Sketchbook/Graphics/Archives/3D/Photography/gallery lightbox/project overlay/CV overlay).
2. HTML body, in DOM order: Hero (`#top`) → category sections `#interior`/`#product`/`#urban` → Stories section `#stories` (links out, not projects) → footer/about (`#about`) → project detail overlay (`#project-overlay`) → CV overlay (`#cv-overlay`) → Sketchbook/Graphics/Archives/3D/Photography full-page overlays (`#sketchbook-page`/`#graphics-page`/`#archives-page`/`#threed-page`/`#photography-page`) → Prints full-page overlay (`#prints-page`) → shared gallery lightbox (`#gallery-lightbox`).
3. `<script>` blocks, roughly: shared utilities + `PROJECTS` data → project overlay logic → CV overlay logic → hero nav/cursor logic → Sketchbook logic → Graphics wall logic → Archives logic → Photography logic → Prints logic → 3D globe logic → hero drawing logic → gallery lightbox logic.

Use `grep -n` for the `<!-- ===...=== -->` HTML banners and `// ---------- NAME ----------` JS banners to navigate — they mark each feature zone.

### Hash routing pattern

Every full-page overlay (project detail, CV, Sketchbook, Graphics, Archives, 3D, Photography) follows the same hand-written sync-function pattern: on `hashchange` (and once on load), if the hash matches, add `.show` + `aria-hidden="false"` + a body scroll-lock class + `scrollTo(0,0)`; else remove all three. **Any new overlay must pair the scroll-lock class `add` with a `remove` in the same function's else-branch** — omitting the `remove` was a real, previously-fixed bug (the old Tales/Photography pages leaked their scroll-lock classes, which tied in CSS specificity with the permanent `body.unlocked` rule and — because it appeared later in stylesheet order — won, permanently breaking page scroll after the overlay was opened once). Sketchbook (`sb-open`) always did this correctly; use it as the reference implementation. Current scroll-lock classes, all correctly paired: `sb-open` (Sketchbook), `gr-open` (Graphics), `ar-open` (Archives), `gb-open` (3D), `cam-open` (Photography), `pr-open` (Prints), `lb-open` (gallery lightbox).

Recognized hashes: `''`, `#interior`, `#product`, `#urban`, `#stories`, `#{project-key}` (any `PROJECTS` key), `#cv`, `#sketchbook`, `#graphics`, `#archives`, `#threed`, `#photography`, `#prints`. `syncProjectOverlay()` no-ops safely on unrecognized hashes.

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

Images are referenced by predictable paths under `renders/`, managed by the user outside the codebase. JS **probes** for files (`probeImage()`/`probeVideo()`, tries `Image()`/loads and resolves true/false) and silently stops on a miss — there is no manifest, so a missing or misnamed file just fails to appear with no error. If "an image isn't showing up," check the filename/number sequence first, not the code.

| What | Path pattern |
|---|---|
| Project gallery images | `renders/{project-key}-{n}.{ext}`, sequential from 1, no gaps |
| Project cover (optional) | `renders/{project-key}-cover.{ext}` |
| Interior/Product/Urban row thumbnails | `renders/{project-key}.png` (falls back to `.jpg` once, then to a plain color swatch — `handleRowImgError()`) |
| Stories row thumbnails | none — Archives/Sketchbook/Graphics/3D/Photography rows use a plain `--swatch-color` dot + inline SVG icon, no image file |
| Sketchbook sketches | `renders/sketchbook-{n}.{ext}`, sequential, cap 24 |
| Graphics wall posters | `renders/graphics-{n}.{ext}`, plus an optional `renders/graphics-{n}-back.{ext}` per piece for its flip side |
| Photography albums | `renders/photography-{1,2}-{n}.{ext}`, sequential, own bespoke grid/viewer (see below) |
| Prints booklets | Cover: `renders/prints-{n}-cover.{ext}` (booklet `n`, sequential, cap 12); interior pages: `renders/prints-{n}-{p}.{ext}`, sequential per booklet, cap 150 (see `getBookletPages()` below - a scanned book can easily blow past the shared 24-slot gallery cap) |

Accepted extensions: `.jpg`/`.jpeg`/`.png` everywhere (`GALLERY_EXTENSIONS`); some gallery slots also accept `.mp4` (`GALLERY_VIDEO_EXTENSIONS`).

### Shared systems (reuse these, don't reimplement)

- **`loadGallery()` / `renderGalleryTiles()` / `probeImage()` / `probeVideo()`** — the horizontally-scrollable filmstrip gallery, used by project pages. Photography's camera-screen view reuses the same `probeImage`/`probeVideo` file-detection convention but renders its own bespoke grid + viewer, deliberately *not* the shared gallery/lightbox — that keeps its permanent digital-screen filter effect (`.cam-filter`) scoped to just that page.
- **`openLightboxFromGallery(galleryEl, startIndex)`** (`window.openLightboxFromGallery`) — shared full-screen lightbox; reads `.has-img` tiles from whatever gallery element is passed, so it works for any gallery with no extra config. Keyboard arrows, backdrop click to close, touch swipe (40px threshold, horizontal-only). Also reused by the 3D globe: clicking a tile explodes it into this same lightbox, and closing the lightbox pulls the sphere back together.
- **`.cat-preview` + `showPreviewFor()`/`hidePreview()`** — hover/tap project-row preview near the section `<h2>`. On touch, includes a `scrollIntoView({block:'center'})` call on first tap so the preview isn't left off-screen above the viewport on long lists — preserve this if reworking.
- **`markProjectVisited(key)`** — session-only (`sessionStorage`) visited-row indicator (`.row-visited`), resets each fresh visit.
- **`getCoverSrc(key)`** — Prints reuses this unchanged, keyed `prints-{n}`, for each booklet's cover. Its interior pages do *not* go through `getGalleryItems()` (24-slot cap, sized for a small project gallery) - Prints has its own `getBookletPages(n)` in the Prints script block, which calls the lower-level `resolveGalleryItem()` directly up to `PAGE_MAX_CHECK` (150), cached the same way. Don't raise `GALLERY_MAX_CHECK` itself to fit bigger booklets - that would add probing overhead to every other gallery on the site for a need that's specific to Prints.

### The five Stories sub-pages are intentionally NOT a shared template

Each has its own distinct visual language by design:
- **Sketchbook** (`#sketchbook-page`) — dark canvas, cursor-following spotlight (`.sb-spotlight`), scattered/rotated sketch images, click-to-focus zoom, no drop-shadow (deliberate). Scroll-lock: `sb-open`.
- **Graphics** (`#graphics-page`) — "investigation wall": posters pinned at angles with red string between them (`#grStrings` SVG), revealed only where an eased, cursor-following spotlight (`#grSpotlight`) falls — same interaction model as Sketchbook's spotlight, applied to a wall instead of a scatter. Click a poster to focus/zoom. Posters come from `renders/graphics-{n}`; shows placeholder prints until real ones are dropped in. Scroll-lock: `gr-open`.
- **Archives** (`#archives-page`) — formerly a separate "Tales" bookshelf; Tales and a new "Reflections" (design-history essays) were merged into one typewriter-style reader. Data lives in one `ARCHIVES` array of `{section, desc, items: [{title, sub?, blocks, imagesKey?}]}` groups (flattened into `FLAT_ITEMS`); picking an item from the `#arList` index types its `blocks` out character-by-character into `#arTyped` (click to skip the animation), styled like a typed page with a photo-corner accent (`#arCorner`). The old bookshelf/page-turning/`paginateBlocks()` engine is gone. Title/hint still use `8vw` left offset (see Design system). Scroll-lock: `ar-open`.
- **3D** (`#threed-page`) — a sketch-globe: tiles wrapping a CSS 3D sphere (`#gbSphere`) that free-spins (yaw + pitch physics, eases back to a steady autospin) and responds to drag. Which tile was tapped is worked out geometrically (projecting each tile's current position to screen space and picking the nearest front-facing one) rather than from the DOM event target, since native hit-testing on `preserve-3d` layers is unreliable across browsers and the globe is moving under the cursor. Clicking a tile explodes it into the shared `#gallery-lightbox`; closing the lightbox re-forms the sphere. Falls back to a flat grid (`#gbFlatGallery`) when needed. Scroll-lock: `gb-open`.
- **Photography** (`#photography-page`) — a "camera screen" page (`.cam-bezel`/`#camScreen`) showing a grid of shots per album (`ALBUMS` array, currently `photography-1`/`photography-2`) with a shot counter readout (`IMG 001 / 010`). Opens its own bespoke viewer (`#camViewer`, prev/next, video support) rather than the shared gallery-lightbox, so its permanent digital-screen filter effect stays scoped to this page only. Scroll-lock: `cam-open`.

### Hero drawing feature (private, client-only)

The hero (`#heroCanvas`, toggled via `.hero.drawing`) lets visitors doodle over the logo with pen/eraser. Strokes save only to that browser's own `localStorage` (`jeame-hero-drawing-v1`) — nothing is ever sent anywhere, no cross-device/visitor sharing. Keep this purely client-side if modified.

### Prints (hero-only entry point, not a Stories page)

`#prints-page`, reached only via the `.side-tab` ("Prints") sticking out from the right edge of the hero — deliberately **not** listed in the Stories section, unlike the five sub-pages above. The table surface (`.pr-table`) is a real photo, `renders/textures/prints-table.jpg` (a dedicated non-probed decorative asset, not part of the numbered `renders/{key}-{n}` content convention — replace this one file directly to change the table look), `background-size:cover` under a faint vignette. Booklet covers are scattered on top at random position + tilt (same forgiving scatter/collision-avoidance approach as Sketchbook).

Clicking a booklet doesn't just show the reader - `flyToReader()` clones the clicked cover into a `position:fixed` `.pr-fly-clone`, animates it from the booklet's exact table position to the centred reader spot (`computeTargetRect()`, matching `.pr-page-wrap`'s own `min(72vw,620px) × 80vh` box), then swaps in the real `#prReader` once that animation *and* the page fetch have both resolved (`Promise.all`). Once open, `#prReader` is a bespoke page-flip viewer: prev/next/arrow-keys/swipe (same swipe pattern as Photography's `#camViewer`) drive `flipTo(index, dir)`, a CSS 3D transform flip — `#prFlip`'s single face is the *outgoing* page, `backface-visibility:hidden`, rotated via `transform-origin:left|right` depending on direction; the static `#prPageImg` underneath is switched to the destination page *before* the rotation starts, so it's already correct once the flipping face turns past 90° and disappears. A `flipping` guard makes rapid clicks a no-op mid-turn rather than queuing or overlapping animations. Booklet 1 is Cem's real printed portfolio (`forwebsite.pdf`, 44 PDF pages → rasterized 150dpi → page 1 = cover, page 44 = back cover, pages 2–43 are two-up spreads split down the middle into 84 individual pages — all exactly square, since the source PDF's spread pages are exactly 2× the cover's width with no gutter). Booklet data comes from `getCoverSrc()` (cover) and `getBookletPages()` (pages, see Shared systems above) keyed `prints-{n}` — see the image-loading table above — so an as-yet-unfilled booklet slot just doesn't appear, and zero booklets shows the "Booklets coming soon" empty state. Scroll-lock: `pr-open`.

## Mobile-specific behavior

`isTouchDevice` (`matchMedia('(hover:none)')`, `ontouchstart`, `maxTouchPoints`) gates two-tap-to-navigate patterns: hero shape nav (first tap arms, second navigates) and project row previews (first tap shows preview, second tap on the same row navigates). Don't bind hover handlers unconditionally — synthetic mouseover/mouseout fires after real taps on touchscreens and can cause flicker if not guarded by `isTouchDevice`.

## Known placeholder/incomplete content (not bugs)

- **Graphics** and **3D**: no longer "coming soon" — both are fully built (investigation-wall and sketch-globe respectively, see above) and have renders in place.
- **Archives**: "The Scarf" (under "Tales from a Young Man's Old Life") and the three "Reflections" essays (on "Design Emergency", on Flores & Prats, on Giovanni Hänninen) are live entries in the `ARCHIVES` array — verify with Cem whether the current text is final copy or still a draft before treating it as placeholder either way.
- **Photography album titles/keys** ("Album One"/"Album Two", `photography-1`/`photography-2`): placeholder naming, real shoot titles not supplied yet.
- **Sketchbook**: shows an empty-state message until sketch images are present in `renders/` (22 processed pages, `sketchbook-1.jpg`…`sketchbook-22.jpg`, are committed and live; `sketchbook-23.jpg`…`-27.jpg` exist in the working folder but are intentionally left untracked — don't add them to the deploy repo without being explicitly asked).
- **Prints**: booklet 1 (`prints-1-*`) is live, Cem's real portfolio PDF, split into 86 individual pages (cover + 84 interior + back cover) — not a placeholder. Additional booklets can be added the same way, up to `MAX_BOOKLETS` (12).

These degrade gracefully by design per the image-probing convention above — don't "fix" them without new content from Cem.
