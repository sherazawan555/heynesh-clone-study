# Build Prompt — Pixel-Perfect Clone of heynesh.com

Paste everything below the line into Claude Code, working in `potfolio-clone1/`.

---

Build a pixel-perfect, animation-exact clone of **heynesh.com** (Nenad Popadic / NESH® — Webflow developer portfolio) as a standalone static site in this folder. This is a study/replica build.

## Ground truth — read these first

`reference/` in this folder contains the **actual production source**, captured live. Do not guess at anything these files answer:

| File | What it is |
|---|---|
| `reference/animation-engine.js` | The site's real custom animation engine ("OPTIMIZED ANIMATION ENGINE v2.5", 3308 lines). **This is the spec for every animation.** Port it essentially as-is. |
| `reference/head-inline-styles.css` | The site's hand-written `<style>` block from `<head>` — **not** in the Webflow stylesheet. Holds the first-paint hiding, gradient headings, marquee, digit-counter CSS, and most hover states. Do not skip this file. |
| `reference/heynesh-live.html` | Full live DOM — every section, class name, `data-*` attribute, copy string, asset URL |
| `reference/nesh-webflow.css` | The compiled Webflow stylesheet — all tokens, breakpoints, component styles |
| `reference/load2.jpg` | Contact sheet of the **preloader/intro** at 4fps — visual ground truth for the opening |
| `reference/scroll_01/02.jpg` | Contact sheets of the full page scroll at 1fps |
| `reference/load_sheet.jpg` | First 6s (recorder setup — site load begins ~5.5s in) |

When `animation-engine.js` and any description below disagree, **the file wins**.

## Live-verified via Playwright (2026-07-29) — corrections to the source-only pass

Everything above was derived from static source. This section is from actually opening https://heynesh.com in a real browser at 1440×900 and 390×844, scrolling through every section, and reading computed layout. Where this contradicts anything above, **trust this section** — it's observed, not inferred.

**The "sidebar" is a full persistent panel, not just a logo.** After the intro, a fixed left column stays on screen for the *entire* rest of the page (confirmed present, unchanged, at every scroll depth down to the footer). Top to bottom it contains, in order:
1. Yellow `NESH®` logo pill + two small square social icons (X, LinkedIn) to its right
2. Four-line description paragraph ("Working closely with your team to deliver Webflow builds...")
3. A stat row: `80+ Projects` | `7+ Years of experience`, separated by a vertical divider
4. Full vertical nav — 7 items (Home, About Me, Projects, What You Get, Services, Clients, FAQ), each a pill with an icon + label. **The current section's pill is solid yellow** — this updates live as you scroll (confirmed: Home → About Me → Projects → What You Get, matching scroll position exactly, i.e. it's ScrollTrigger-driven active-state tracking, not just a hash match)
5. A row of client logos — **and this is an infinite marquee**: the visible logos changed between two screenshots taken seconds apart at the same scroll position with no page interaction (`puck / DI 1910 / alosant` → `CURRI / Ωomicron / puck` → `Happy Ring / INVERT / SemiconBio`), confirming `animation: marquee-move 25s linear infinite` is live and continuous, not hover-triggered
6. An email chip (`nenad@popadic.co`) with a copy icon
7. A persistent yellow `Book a Call` button, always the last element, always visible

**Dark theme is real and exactly scoped to `#work`.** Confirmed by screenshot: the moment the work section fills the viewport, the entire sidebar inverts — panel backgrounds go charcoal/near-black, nav pills go dark gray, text goes off-white — then reverts to the light stone theme immediately after. Exactly one section triggers it, as the source implied.

**New section not fully captured before — the "What You Get" capabilities statement has a scroll-scrubbed word reveal.** It's one long sentence, big bold serif-adjacent sans, broken by 5 inline icon-pill tokens sitting mid-sentence at word boundaries (small rounded rect, pale bg, yellow icon glyph + tiny dot). The exact structure, confirmed at rest:

> "Strategy, precision, and development combined - turning **[chart-icon pill]** your vision into a powerful digital experience **[gear-icon pill]** that feels **[??-icon pill]** effortless. **[??-icon pill]**"

(5 pills total interleaved through the sentence — capture exact icon glyphs and pill positions from `heynesh-live.html` rather than this paraphrase.) Mid-scroll, the *tail* of the sentence is heavily faded (near-invisible, low-opacity) while the portion already scrolled past is fully opaque black — this is a per-word/per-line opacity scrub tied to scroll position, distinct from the capability-card hover engine already documented. Look for a SplitText line/word split feeding a ScrollTrigger-scrubbed opacity tween in `animation-engine.js` and make sure it's ported — this was easy to miss because it only resolves correctly mid-scroll, never at rest.

**Services/pricing section is one solid card, not three floating cards.** The three tiers (`$3,000/30hrs` Ongoing Support, `$5,000` Starter Build, `Book a Call` Custom Project) sit inside a single continuous rounded panel with internal vertical dividers between columns — not three separate elevated cards with gaps. Each column: icon badge (yellow rounded square) + title, price, description, bullet list with circular ring bullet icons, then a smaller "for X" qualifier line at the bottom in a lighter weight.

**CTA section ("Transform Your Webflow Experience Journey") has a two-tone heading treatment** I hadn't captured: "Transform Your Webflow" renders solid black/dark, but "Experience Journey" directly below it renders in a much lower-contrast tint (near page-background color, almost a ghost/outline effect) — not the same gradient as the `.h2-style` headings elsewhere. Below the heading sits a small circular avatar photo + a chat-bubble-shaped pill reading "Have something in mind?" + a yellow "Let's Talk" button, arranged like a chat prompt.

**Testimonial cards match the CSS exactly, confirmed live**: yellow `"` quote-mark badge top-right of each card, headline, body copy, then avatar photo + name + role + linked company at the bottom. Swiper pagination dashes sit top-right of the section, first bullet wide and yellow (`.swiper-bullet.is-active`), rest short and pale — pixel-matches the `width: 2.5vw; background: #ffff23` rule pulled from source.

**Footer bookends with a second giant yellow NESH wordmark** above the FAQ heading — mirrors the hero's giant background type. FAQ is a clean 2-column grid of question rows, each ending in a `+` icon (confirmed via source it rotates to become `×`/`−` on open).

**Mobile (390×844) confirmed to skip the intro entirely**, exactly as the CSS reset predicted: on load the logo pill is already docked top-left at final size, next to a persistent yellow `Book a Call` and a 4-dot grid icon (opens the mobile nav). No FLIP animation plays — it doesn't need to, since there's no sidebar to animate into.

## The intro, frame-by-frame — captured by pausing GSAP's own clock, not wall time

Wall-clock screenshots can't catch a 3.5s animation because tool round-trips are slower than that. Fixed it properly instead: injected an init script (`page.addInitScript`) that intercepts `gsap.timeline()` at the moment the library defines it and force-pauses every timeline at `t=0` the instant it's created — before the site's own code can advance it a single frame. Confirmed the real intro timeline: **index 2 of the timelines created on load, total duration 3.53s, with a `heroReveal` label at exactly `t=1.4`** — this is the literal GSAP timeline the live site runs, scrubbed with `tl.time(x)`, not a recreation. Below is what's actually on screen at each mark, all at 1440×900:

| t | What's on screen |
|---|---|
| **0.00** | Flat stone background. Nothing else — logo is fully off-screen right (`translate(110%)`), hero is otherwise `visibility:hidden`. Matches the CSS-predicted first paint exactly. |
| **0.50** | The wordmark is mid-slide-in from the right edge — at this instant only "NE" plus the start of "SH" is within the viewport, letters entering as one solid block, no per-letter stagger visible yet at the word level. |
| **1.00** | "NESH" fully centered and filling the frame, letters at rest — this is the moment the logo-travel tween (`duration: 1, ease: power3.inOut`) completes. |
| **1.40** (`heroReveal` label) | Still just centered NESH — this is the label instant, before any of the tweens keyed off it have produced visible movement yet. |
| **1.80** | Profile photo is **blurring into focus through the yellow letters** (filter blur tween resolving, photo and NESH overlap with the photo semi-transparent/blurred), and the "Webflow," headline text is visible but still blurred underneath. **Nav is still masked here** — each `.hero-navigation-link` inner is at `translateY(18px)` behind its `overflow:hidden` line-mask, because the nav reveal is keyed to `heroReveal+=0.6` (= t=2.0). Verified by measuring computed transforms on live and clone at this exact time — both report `matrix(1, 0, 0, 1, 0, 18)`. (An earlier capture of this frame appeared to show legible nav text; that was a race between SplitText init and the freeze, not stable behavior — unsplit links briefly render as plain text. Don't build to it.) |
| **2.20** | Photo fully sharp/in-focus. Heading "Webflow, Applied Differently." still visibly blurred (filter tween not yet resolved). One capability-list card (bottom-left area) is a soft blurred yellow ghost blob — not yet positioned or legible; this is the card's own blur-in tween mid-flight, separate from the headline's. |
| **2.65** | Both stat cards (`80+ Projects`, `7+ Years of experience`) are sharp and settled on the left. The right-side capability-list card is still a faint unresolved blur ghost. CTA buttons (`Book a Call`, `About Me`) not yet visible — confirms button reveal (`heroReveal+=1.25` / `+=1.33`) fires after the cards. |
| **3.53** (end) | Fully settled — pixel-identical to the resting-state screenshot taken via normal navigation. Confirms 3.53s is genuinely the complete intro duration, not just where I stopped looking. |

Sequence to build from, in order: **wordmark slides in from off-screen right → centers and fills frame → (heroReveal) → photo blur-resolves + nav text fades in flat (no chip bg) → headline blur-resolves → stat cards blur-resolve → capability card blur-resolves → buttons fade/scale/blur in last.** The blur-focus resolve (not a simple fade) applied to the photo, headline, and cards is the detail every static-source pass would miss — it's a `filter: blur(Npx) → blur(0px)` tween, not an opacity tween, and it's visibly still mid-blur at 1.8s–2.65s.

Screenshots backing this table live in `reference/intro-frames/` as `seq-00-t0.00.png` through `seq-07-t3.53-end.png` — use them as literal frame references, not just the description above.

## Scroll choreography — captured deterministically, not by wall clock

Everything past the intro is `ScrollTrigger`-scrubbed, keyed to `scrollY`, not time — which makes it *more* reliable to capture than the intro: set `window.scrollTo(0, y)`, force `window.ScrollTrigger.update()`, screenshot. No animation frame is ever missed because nothing is playing on its own clock; it only moves when scroll position moves. Read the exact trigger ranges straight from the live `ScrollTrigger.getAll()` registry (all `trigger: hero`, values in px at 1440×900, hero total height 2700px):

```
logo FLIP:            start 0,   end 1188   (= 44% of hero height, matches data-flip-end="44% top")
nav-webflow-*:         start 0,   end 1080   (= 40%)
nav-menu-item (×7):    staggered 189→810, 162→837, 135→864, 108→891, 81→918, 54→945, 27→972
```
These are the literal numbers the production ScrollTrigger instances report — not recomputed from CSS percentages, read directly off the running page. Screenshots for everything below are in `reference/scroll-frames/`.

**Logo/sidebar FLIP journey (`flip-00` → `flip-04`, scrollY 0 → 1188):**
- **y=0**: baseline hero, exactly as the settled intro state.
- **y=300**: photo is visibly **blurring out** (opposite direction from the intro's blur-in — this is a *second*, separate blur tween keyed to scroll, not a reverse-play of the intro). A **® mark** has appeared next to the H, riding the same FLIP range as the logo (this is `.nesh-copyright-icon`, confirmed from source, now confirmed visually). Nav items are already font-size-shrinking and drifting to individually-staggered vertical offsets — at this single scroll position, `ABOUT ME`/`PROJECTS` etc. are visibly NOT aligned with each other, proving the per-item stagger is real and not just a source artifact.
- **y=600**: logo has shrunk to roughly half its full-bleed width, still transitioning color/position; nav items are far enough along that a stat card (`80+ Projects`) visibly **overlaps and crosses through** the nav `PROJECTS` label — two independent FLIP tracks passing through the same screen region mid-transition. Photo now heavily blurred, near its blur peak.
- **y=900**: sidebar shape has basically resolved (vertical nav list, stat row, logo pill) but **not every nav item has its pill background yet** — `HOME` through `SERVICES` have their gray chip, `CLIENTS`/`FAQ` are still bare text, confirming the per-item background-fade lags behind the position FLIP. Photo still a huge blurred presence behind the emerging About-section heading — hero and about content visibly overlap here.
- **y=1188 (FLIP end)**: sidebar is now **pixel-identical to its final resting form** — full chip backgrounds, active-state yellow on `ABOUT ME`. The blurred photo is still there as a large soft background wash behind the About heading; it does not disappear at the same moment the FLIP completes — it fades out over its own separate, longer scroll range (still faintly present even at y=2000/2700). Build the photo-fade as its own independent scroll-scrubbed opacity/blur tween on a longer range than the FLIP, not tied to the same 0–1188 window.

**About section reveals a scroll-driven journey timeline** (`flip-05`, `flip-06` — not documented anywhere in the static source pass, easy to miss entirely): a vertical zigzag sequence of year-stamped cards (`'20 First freelance steps`, `'22 Leveling up`, `'25 From trust to referrals`, etc.), each styled like a social post — big yellow year, bold title, description, small circular avatar/logo, `@handle`, `Xyears ago`, a `Read more` button — alternating left/right down the page, connected by a single hand-drawn-style curved SVG line with small circular node markers at each card's connection point. Build this as an SVG path (`stroke`, not straight lines — visibly curves) with dot markers positioned at each card, revealed progressively as you scroll past each one.

**Work section is pin + horizontal scrub, with heading staying fixed** (`work-00` → `work-02`, scrollY 4649 → 8100, work section spans 3600px = 400vh): confirmed the `SELECTED WORK` / `Built in Webflow, Made to Perform` heading block does **not move at all** between scroll positions while the card track visibly slides — cards `01/02` at y=4649 become `03/04/05` at y=5849 become `08/09` at y=8100. This is a pinned section: the heading is pinned in place and only `.work-track` translates horizontally, driven by the vertical scroll distance. Do not animate the heading — only the card track.

**Dark-theme transition is a smooth scrub, not a toggle** (`work-02`, y=8100, the work→overview boundary): captured mid-transition — the dark work track is still visible pinned at the top of the viewport while the light-themed "What You Get" heading is already rising into view beneath it, and the sidebar itself is visibly **half-dark, half-light** in the same frame (dark background near the top NESH logo pill, transitioning to the light stone tone lower down). This confirms the `data-theme="dark"` swap on `.work_section` is animated (a scrubbed color/background tween across the section boundary), not a hard CSS class swap at a single scroll pixel.

Frame-by-frame files for all of the above: `reference/scroll-frames/flip-00-y0000.png` through `flip-06-y2700.png` (hero FLIP + About timeline), `work-00-y4649-start.png` / `work-01-y5849.png` / `work-02-y8100-end.png` (Work pin/scrub + theme transition). Use them as literal references alongside `animation-engine.js`.

## Stack — match exactly

```
GSAP 3.15.0 + ScrollTrigger, ScrollSmoother, SplitText, Flip,
             MotionPathPlugin, DrawSVGPlugin, ScrollToPlugin
Lenis 1.1.18   (lerp 0.1, duration 0.4, smoothWheel true)
Swiper 11      (testimonials only)
```
Plain HTML/CSS/JS — no framework. The original is Webflow-output; reproduce the markup structure and class names from `heynesh-live.html` so the ported engine's selectors resolve unchanged.

## Design tokens

- Page bg `rgb(213,207,190)` warm stone · dark sections `#0A0A0A` · dark-section text `#F8F7F3`
- Accent yellow `#FFFF23` · nav light chip `#EBEADA` · nav light bg `rgba(223,222,206,0.80)`
- Nav dark bg `rgba(29,29,29,0.60)` · dark chip `rgba(57,57,57,0.90)`
- Body: **PP Neue Montreal** (Book/Medium/Bold) — 14.4px/23.04px, weight 400
- Display: **TR 3 A** (Regular/Medium/Bold) — H2 65.95px/65.95px w500
- `.hero-heading` is fluid: `clamp(3rem, 5.3vw, 5.5rem)` desktop, `clamp(2.25rem, 10.91vw + .068rem, 5.23rem)` mobile, with `kerning: none`

**Section headings are gradient-filled, not flat black** — easy to miss and very visible:
```css
.h2-style,       .h2-style .line       { background: linear-gradient(266deg, #3d3d3d 11.86%, #000 92.59%); }
.h2-style-white, .h2-style-white .line { background: linear-gradient(89deg, #d5d5d5 7.42%, #fff 95.11%); }
/* both: background-clip: text; -webkit-text-fill-color: transparent; */
```
The gradient is declared on **both** the parent and each `.line`, because SplitText splits them. `StyleEngine` detects `background-clip: text` on the parent post-split and neutralizes the parent's own paint (`background: none; -webkit-text-fill-color: transparent`) to kill a doubled ghost. Port that guard.

Real font files (fetch and self-host):
```
.../691d8a7bc4e9f35b184ed616_ppneuemontreal-book.woff2
.../6942a13115cd4b3a65a5972f_ppneuemontreal-medium.woff2
.../691d8a7b11b33d4b469b6f1c_ppneuemontreal-bold.woff2
.../691d8ab1156537b9520e5061_tr3a-Regular.woff2
.../691d8ab15c6a560ca0833ae1_tr3a-Medium.woff2
.../691d8ab14cbd4a1cf9811e7a_tr3a-Bold.woff2
```
Base: `https://cdn.prod.website-files.com/691d7c9f14d0280ebe2d4108/`

## Interaction behaviour — measured on both live and clone

All values below were read from the running page (computed styles / live Swiper instance), on the live site and the built clone, and matched exactly. Screenshots in `reference/clone-verification/`.

**Work-card group hover** (`@media (min-width: 992px)`) — hovering any one card:
| | hovered card | every other card |
|---|---|---|
| `.work-image-overlay` opacity | `0` | `1` |
| `.work-card-arrow-wrap` background | `rgb(255,255,35)` | `rgb(47,47,47)` |
| `.work-card-arrow-wrap` color | `rgb(0,0,0)` | `rgb(140,140,140)` |

Hovering also **loads and plays the card's transparent foreground video** (`.work-video`) over the still background — the card visibly switches from its static poster to animated product footage. See "Work-card videos" below.

**FAQ accordion** — clicking `.faq-toggle` adds `w--open` to both the toggle and its `.faq-list`; `.faq-answer-wrap` animates `grid-template-rows: 0fr → 1fr` (resolved to ~187px for the first item); the `+` becomes a `−` via `.faq-icon-v-line { transform: rotate(90deg) }` (computed `matrix(0, 1, -1, 0, 0, 0)`). Only the clicked item opens — the others stay `+`.

**Testimonial Swiper** — identical config on live and clone:
```
slides: 8   slidesPerView: 1   spaceBetween: 14   speed: 500
loop: false   autoplay.enabled: false   bullets: 8
```
It is **drag-only — there is no autoplay**; don't add any. A real mouse drag advances `activeIndex 0 → 1` and moves `.swiper-bullet.is-active` in step. While dragging over the track, a **yellow circular "DRAG" badge replaces the mouse pointer** — that's `.drag-wrap` (present in the DOM on both) made possible by `.swiper-wrapper { cursor: none !important }`.

**Work-card videos** — two stacked layers per card, driven by one inline script (not by `animation-engine.js`):
- `.work-bg` — background loop; full URL sits in `data-src`; autoplays when the card enters the viewport (IntersectionObserver, `rootMargin: '300px'`).
- `.work-video` — transparent foreground; `data-webm` / `data-mov` hold **bare filenames**, and the script builds the src as `FG_BASE + encodeURIComponent(file)`. Safari gets `.mov`, everyone else `.webm`. On desktop (`hover: hover`) it loads and plays on `mouseenter` and resets to `currentTime = 0` on `mouseleave`; without hover it simply autoplays.

`FG_BASE` is a hard-coded CDN folder string in that inline script. If you localise assets you must repoint it — and be careful with blanket find-and-replace across the HTML, since it is easy to clobber this constant by accident (that mistake produces 404s only on the work section, and only on hover).

## Layout geometry — exact declared values

These come from the production stylesheet, not from eyeballing. Several are load-bearing: the tall hero and work section are what give the scrub animations their scroll distance, so changing them silently breaks the timing.

```css
.hero            { min-height: 300vh; padding-bottom: 180vh; inset: 0 0 auto;
                   z-index: 5; pointer-events: none; }
.work_section    { min-height: 400vh; pointer-events: auto; }
.work-track-wrap { padding-left: 55%; display: flex; }
.work-card       { width: clamp(17rem, 27vw, 30rem); border-radius: .83vw;
                   overflow: hidden; transition: opacity .4s ease; }
.swiper          { width: 33.13vw; align-self: flex-start; }
.footer-logo     { aspect-ratio: 3.85; width: 100%; margin-bottom: 5vw; }
.nav-container   { opacity: 0; gap: 16px; grid-template-columns: 1fr;
                   grid-template-rows: repeat(5, minmax(100px, 1fr)) auto; }
```

`.nav-container` ships at `opacity: 0` — the Preloader is what reveals it at `t=1.4`. If you skip the preloader the sidebar never appears.

Breakpoints (Webflow defaults): `max-width: 991px`, `max-width: 767px`, `max-width: 479px`, plus `min-width: 768px`. The 768px line is the same threshold every JS module guards on — keep CSS and JS agreeing.

Custom properties:
```css
--yellow: #ffff23;  --black: black;  --white: white;
--tra: "Tr 3 A", Arial, sans-serif;
--card-hover--card-text: 0;              /* hover-driven, 0 → 1 */
--card-hover--card-icon-size: 1;         /* → per-card data-var-hover */
--card-hover--card-padding-top-bottom: 1;/* → 1.35 desktop / 2.61 mobile */
--card-hover--card-padding-left-right: 1;/* → 1.25 desktop / 1.35 mobile */
--card-hover--scale: calc(100vw / 375);  /* fluid unit, 375px design base */
```
Capability-card hover animates these custom properties, not discrete CSS props — that's why the padding and icon growth stay perfectly in sync.

## Assets

`heynesh-live.html` lists **57** production assets on that same CDN base (portrait `.avif`, project stills, client logos `.svg`, 8 testimonial headshots `.avif`, favicon, OG image). Download all of them into `assets/` and rewrite references to local paths. Keep AVIF; add JPEG fallbacks only if a target browser needs them.

**Project cards are video, not images** — this is easy to get wrong. Each `.work-card` has:
- `.work-bg` — background MP4, full URL in `data-src`, autoplays when the card intersects
- `.work-video` — **transparent-background** foreground video, `data-webm` / `data-mov`, filename only, resolved against `https://f1-assets.b-cdn.net/nesh-work/Portfolio%20Work/`. Safari gets `.mov` (HEVC alpha), everyone else `.webm` (VP9 alpha).

Loading rules (bottom inline script of `heynesh-live.html`): `IntersectionObserver` with `rootMargin: '300px'` loads + plays bg and warms fg; desktop (`hover: hover`) plays fg on `mouseenter` and resets `currentTime = 0` on `mouseleave`; touch devices autoplay fg instead. Off-screen pauses both.

## First paint — before any JS runs

Get this wrong and the intro flashes the whole hero for a frame before hiding it. There is **no covering overlay or spinner**; the "loading screen" is the real page with almost everything pre-hidden in CSS. From `head-inline-styles.css`:

```css
/* Hide entrance elements initially */
.profile-img-item, .hero-navigation-link, .hero-navigation-sep,
.hero-card-3, .hero-cards-left, .hero-left-text,
.hero-right-text, .hero-buttons-wrap { opacity: 0; visibility: hidden; }

.hero-heading { visibility: hidden; }   /* visibility only — no opacity */
```
Plus, from the Webflow stylesheet: `.nav-container`, `.hero-card-1`, `.hero-card-2`, `.hero-card-2-shadow`, `.hero-cta-button`, `.nav-top-bg`, `.work-image`, and every `*-ghost` element are all `opacity: 0`.

The one thing visible at first paint is the giant logo, parked off-screen right by CSS — **not** by JS:
```css
.nesh-logo-preload-svg { opacity: 1; transform: translate(110%);
  width: 94.44vw; aspect-ratio: 3.8; color: var(--yellow);
  position: absolute; inset: 0 0 auto; transform-origin: 0 0; }
```
So the true first frame is: flat beige page, yellow NESH just off the right edge. The SVG viewBox is `0 0 1288 338`.

Below 767px the same stylesheet **resets every one of those to `opacity: 1; visibility: visible`** — mobile has no intro at all and paints the finished hero directly. Reproduce that reset; don't rely on the JS guards alone.

## The opening sequence (`Preloader`, engine line ~1095)

Desktop only — **skipped entirely below 768px**. Fires on `window.load` inside a double-`requestAnimationFrame` after `scrollTo(0,0)`, with `history.scrollRestoration = 'manual'`. Timeline `delay: 0.2`.

1. Giant yellow **NESH** SVG starts at `x: window.innerWidth`, vertically centered; letters pre-set `yPercent: 110`
2. `t0` → logo slides to horizontal center, `duration 1, power3.inOut`; letters rise `yPercent: 0`, `duration 1, stagger 0.2, power3.out` (concurrent)
3. `t1` → logo travels to its final resting spot `{x:0, y:0}`, `duration 1, power2.inOut`
4. `t1.4` → `.nav-container` becomes visible; label `heroReveal` is set here
5. `t2` → preloader logo `display:none`, real `.nav-logo-item` restored — a clean handoff

Then the staggered hero reveal, all offsets relative to `heroReveal`:

| Offset | Element | From → To |
|---|---|---|
| `+0` | `.profile-img-item` | `autoAlpha 0, scale .88, blur(20px)`, origin center bottom → 1/1/0, `1.1s power2.out` |
| `+0.3` | hero heading lines | `autoAlpha 0, scale .90, blur(10px)` → 1/1/0, `1.0s, stagger .1` |
| `+0.6` | nav links | mask-wrapped, `yPercent 100 → 0`, `0.4s`; separators `height 0vw → 0.8vw`, `0.2s` |
| `+0.60…+1.20` | webflow trio, experience trio, `.hero-card-3` | `autoAlpha 0, blur(8px)` → 1/0, `0.9s`, **0.1s apart in that order** |
| `+1.25 / +1.33` | `.nav-button` / `.nav-button-secondary` | `autoAlpha 0, scale .94, blur(10px)` → 1/1/0, `0.8s` |
| `+1.65 / +1.75` | `.hero-left-text` / `.hero-right-text` | per-line `yPercent 100, autoAlpha 0, blur(6px)` → 0/1/0, `0.7s, stagger .075` |

Lenis is **stopped for the first 3000ms** on desktop so the intro can't be scrolled through.

## The signature scroll transition (`GhostEngine`, line ~273)

The hero's big elements **morph into the fixed left sidebar** as you scroll — the single most distinctive thing on the site. Desktop only.

It is a hand-rolled FLIP: real sidebar elements are measured against hidden "ghost" twins sitting at the hero positions, then `gsap.fromTo` runs the real element from the ghost's rect back to its own, `scrub: 1`.

**The scroll ranges are per-element, read from `data-flip-*` attributes in the HTML** — the `'900px top'` in `getSettings()` is only a fallback and is never actually used. Every one of these is on `trigger: .hero`:

| Element | start | end |
|---|---|---|
| `.nesh-logo` (the big wordmark → top-left) | `top top` | **`44% top`** |
| `.nav-webflow-bg` / `-icon` / `-text` | `top top` | `40% top` |
| `.nav-experience-bg` / `-numb` / `-text` | `top top` | `40% top` |
| `.nav-button-wrap` | `5% top` | `35% top` |
| `.nav-menu-item` ×7 | **staggered** `7%,6%,4%,5%,3%,2%,1%` | **staggered** `30%,31%,32%,33%,34%,35%,36%` |

That nav-link stagger is the detail that makes the menu **fan** into place instead of moving as one block — each link starts slightly earlier and finishes slightly later than the one above it. Uniform ranges will look obviously wrong.

So the logo's journey is: it finishes the preloader at hero-center, then over the first **44%** of the hero's 300vh scroll it scales and travels up-left into the sidebar slot, `ease: power1.inOut`, `scrub: 1` — meaning it's tied to scroll position, scrubbing backwards if you scroll up. The copyright ® mark rides the same range via the separate phase-3 handler.

Two-phase and the order is load-bearing: **measure every pair on a clean DOM first, then apply.** Measuring after mutation gives wrong rects.

Pairs (`createAll`): `.nav-logo-item .nesh-logo`↔`.nesh-logo-ghost` (logo) · `.nav-button`↔`.hero-cta-button` (logo) · `.nav-button-secondary`↔`.hero-button` (logo) · each `.hero-navigation-link`↔`.hero-link-ghost[data-link-id]` (link) · `.nav-webflow-bg`↔`.hero-card-2-bg` (background) · `.nav-webflow-icon`↔`.hero-webflow-icon` (icon_center) · `.nav-webflow-text`↔`.hero-webflow-projects-text` (text_font) · `.nav-experience-bg`↔`.experience-bg` (background) · `.nav-experience-numb`↔`.experience-number` (icon_center) · `.nav-experience-text`↔`.experience-text` (text_font). `.nesh-copyright-icon` is handled in a separate phase 3 **after** the others, because the logo tween sets `.nav-logo-item` to `position: relative` and shifts it.

Per-type transform rules and the elliptical `border-radius` compensation for `background` (each corner `${ghostR/sX}px ${ghostR/sY}px` so it reads circular after non-uniform scale) are in `applyAnimation` — port them literally.

All deltas divide by `STATE.sidebarScale`, since `Sidebar.scale()` (line ~128) shrinks `.nav-container` to `min(1, (innerHeight - 40) / scrollHeight)` and applies the inverse scale to the profile image and button text.

## Every other behavior — all in the engine file

- **`ProfileImage`** — once the hero is 50% scrolled past, the portrait is reparented to `<body>` as `position: fixed` at its exact current rect (`z-index: 1000`), and reparented back into `.nav-container` on the way up
- **`ThemeSwitcher`** — sidebar chips invert per element when they geometrically overlap any `[data-theme="dark"]` section (`getBoundingClientRect` overlap; `.nav-menu` needs ≥35% overlap, others any). Swaps colored↔white client logos. `0.3s` tweens
- **`HorizontalScroll`** — `.work_section` pins `.work-sticky`; `.work-track` translates by its right-overflow, `scrub: 1`, `invalidateOnRefresh`. `.work-sticky-support` height = section − sticky. Cards start `y:10%, opacity:0, scale:0.6`; already-visible ones stagger `0.1` at `top 80%`, off-screen ones use `containerAnimation` with `start: 'left right'`, `1.1s expo.out`
- **`StyleEngine`** — generic declarative driver for `[data-tl-*]` attributes (`data-tl-type|trigger|start|end|from|to|split|target|once|desktop`). Includes the **odometer year counters**: each digit becomes a 0-9 `.digit-track` pre-set to `y = h*9`, tweened to `-digit*h`, `1.2s power3.out`, `stagger 0.06`
- **`MagneticPositions`** — `[data-origin]` elements are pinned to `[data-connect]` anchors every rAF via `translate3d`, with `data-offset`/`data-anchor-pos` and `-mobile` variants. Loop runs desktop only
- **`CardInteractions`** — About "Read more" popups (`.is-active-card`, one at a time); capability cards hover-expand on desktop via CSS custom props (`--card-hover--card-text`, `…-padding-*`, `…-card-icon-size`, `0.65s power3.out`) with `.gsap-icon-letter` staggering in (`0.1`) and out reversed (`0.05, from:'end'`). Mobile taps clone the card into a centered `.capa-overview-wrap` overlay (`rgba(0,0,0,0.8)`, `0.5s power3.out`, arrow rotates 180°)
- **`CTAAnimation`** — a scripted chat: typing bubble pops (`back.out(1.7)`), dots loop, bubble physically travels to each next grid row as the chat line then the button expand in (`gridTemplateRows 0fr→1fr`), then the typing bubble scales away. `ctaSpeed: 0.728`, fires once at `top 90%`
- **`SwiperInit`** — `slidesPerView: 1`, `spaceBetween: 14`, `speed: 500`, `autoHeight`, `resistanceRatio: 0.85`, custom `.swiper-bullet` / `.is-active` pagination. Custom cursor-following **DRAG** indicator (`.drag-wrap`, `position: fixed`, `z-index: 9999`) appears on `mouseenter`, tracks the pointer, and scales the left/right icons `1.5` / `0.8` based on drag direction
- **`TextReveal`** — `.what_you_get-text` split per-character preserving inline `data-connect` icons; chars scrub from `#E0DFC5 / opacity .1 / y 5` to `black / 1 / 0`, `stagger 0.1`, `start 'top 92%'`, `end 'top 25%'`
- **`ImageTrail`** — 🔑 **the mouse animation.** Hovering the giant footer `.footer-logo` paints a trail of project thumbnails *inside the SVG*: `<image>` nodes appended to `#image-trail-group`, `200×280`, `preserveAspectRatio: 'xMidYMid slice'`, random rotation ±30°, new node every `30` SVG units of travel, max `20` live, FIFO fade at `50ms` intervals starting `100ms` after the pointer stops, `0.3s` opacity transition. Coordinates convert through the SVG `viewBox`. 8 hardcoded image URLs — list is at engine line ~2841. Desktop only
- **`ButtonHover`** — `[data-button-hover]` duplicate-text vertical swap; must detect and reuse `.nav-link-mask-inner` when the Preloader already wrapped that element
- **`Clipboard`** — email hover reveals "Copy to clipboard"; on click → "Email Copied", chip flips to `#FFFF23`, check icon pops
- **`MobileMenu`** — `<768px` only; `.nav-menu-wrap` reveals via `clipPath inset(0% 0% 100% 0%) → 0%`, `0.8s power2.out`; hamburger bars split ±(50 + gapOffset)%
- Anchor links are intercepted for `scrollIntoView({behavior:'smooth'})` and strip the hash via `history.replaceState`

### Details that live only in `head-inline-styles.css`

- **Client-logo marquee** — `.nav-comapny-item { animation: marquee-move 25s linear infinite }`, `translateX(0 → -100%)`
- **Group hover on the project rail** (`@media (min-width: 992px)`) — hovering *any* card raises `.work-image-overlay` to `opacity: 1` on **all** cards and dims every `.work-card-arrow-wrap` to `#2F2F2F / #8C8C8C`; the hovered card alone reverts to `opacity: 0` and `#FFFF23 / black`. Result: hover one card and the rest recede. Card arrows also slide (`.work-card-arrow { margin-left: 100% }`, `.work-card-arrow-2 { margin: 0 }`)
- **`.swiper-wrapper { cursor: none !important }`** — this is what lets the custom DRAG indicator replace the pointer
- **Odometer digits** — `.digit-mask { height: 1em; width: .65em; overflow: hidden }`, `.digit-track` is an absolute flex column, `[data-number-count] { font-variant-numeric: tabular-nums }`
- **Swiper pagination** — `.swiper-bullet.is-active { width: 2.5vw; background: #ffff23 }` (mobile `clamp(1.92rem, 9.6vw, 4.61rem)`); container has `1px solid rgba(255,255,255,.2)`, `border-radius: .56vw`, `overflow: visible !important`
- **FAQ** — `.faq-list.w--open .faq-answer-wrap { grid-template-rows: 1fr }`, `.faq-toggle.w--open .faq-icon-v-line { transform: rotate(90deg) }` (a `+` morphing to `−`), hover `#EDECDA`
- **Active nav state** — `.w--current` gets black text, `~ .nav-item-bg` turns `#FFFF23`; hover turns bg `#C9C8BA` (or `rgba(94,94,94,.5)` in dark mode) and the icon `#ffff23`
- **Popup close** — the two bars counter-rotate `±90°` to white on hover
- **Safari fixes** — `[data-tl-to*="blur"]` and `.hero-profile-img` need `translateZ(0)`, `backface-visibility: hidden`, `perspective: 1000px`, explicit `will-change`. Without these, blur tweens flash white borders in Safari
- `.nav-logo-item { position: relative !important }` and `will-change: width, height, transform` on all FLIP participants

**Mobile is a genuinely different build, not a media query.** These are hard-disabled below 768px: Preloader, GhostEngine, HorizontalScroll, ThemeSwitcher, ImageTrail, ButtonHover, Sidebar auto-scale, MagneticPositions rAF loop. Reproduce the guards, don't just restyle.

Init order (`initAll`, line ~3229) matters — Sidebar → GhostEngine → StyleEngine → HorizontalScroll → ThemeSwitcher → MobileMenu → CardInteractions → ProfileImage → CTAAnimation → Clipboard → ImageTrail → ButtonHover → Swiper → Lenis → TextReveal → ResizeHandler → `ScrollTrigger.refresh()`, then `MagneticPositions` after a 300ms timeout.

## Content

All copy, project entries, testimonials, FAQ, pricing and the JSON-LD `Service` block are in `heynesh-live.html` — **take every string from there verbatim**, do not paraphrase or invent. Nine projects `01`–`09` with capability tag pills and outbound links; eight testimonials with named headshots; eight FAQ items in two columns; three pricing tiers (`$3,000/30hrs`, `$5,000`, Book a Call). CTA → `https://cal.com/nenad-popadic/intro-call`.

### Section skeleton — exact, in document order

Seven `<section>` elements. Note the id/anchor mismatch; it is easy to get wrong:

| `<section>` id | class | theme |
|---|---|---|
| `hero` | `.hero` | — |
| `about` | `.about-section` | — |
| **`work`** | `.work_section` | **`data-theme="dark"`** |
| `overview` | `.what_you_get_section` | — |
| `services` | `.sevice_section` *(sic — misspelled in production; keep it, the CSS depends on it)* | — |
| `webflow_journey` | `.cta_section` | — |
| `testimonial` | `.testimonial_section` | — |

Nav anchors point at `#hero #about #projects #overview #services #testimonial #faq`. But `#projects` and `#faq` are **not** the section ids — they're separate anchor `<div>`s nested inside (the projects section is `id="work"`, and FAQ lives in the footer, not in a `<section>` at all). Reproduce both the section ids and the inner anchor divs or the smooth-scroll nav lands nowhere.

**There is exactly one `data-theme="dark"` element on the page** — `.work_section`. So `ThemeSwitcher` only ever fires as the sidebar crosses the projects rail. Don't mark other sections dark; you'd trigger inversions the original never does.

## Verification

Serve locally and check, in this order:
1. Intro plays exactly as `reference/load2.jpg` shows — NESH sweeps in from the right, letters stagger up, portrait blurs in behind, then nav/cards/buttons/paragraphs cascade
2. Scrolling the hero morphs logo, buttons, nav links and both stat cards into the sidebar with no jump or flash
3. Sidebar chips invert over dark sections and revert cleanly
4. Project rail scrolls horizontally while pinned; card videos autoplay, and hovering swaps in the transparent foreground video
5. Footer logo hover paints the rotating image trail and it FIFO-fades when the pointer stops
6. Hard refresh mid-page starts at top (`scrollRestoration: 'manual'`)
7. Resize across the 768px line without console errors — `ResizeHandler` rebuilds ghosts and magnetic pairs
8. `prefers-reduced-motion` reduces motion (the original does **not** honor this — add it)

Build it section by section, verifying each against the reference files before moving on.
