# heynesh.com — study replica

A local, self-contained replica of [heynesh.com](https://heynesh.com) (the portfolio of Nenad Popadic / NESH®), rebuilt for study of its scroll and animation architecture.

**Private repository.** Not for redistribution or deployment — see [Attribution & licensing](#attribution--licensing).

## Running it

```bash
node serve.js
# http://localhost:8899
```

No build step and no network access required — every asset is local.

## What's here

| Path | |
|---|---|
| `index.html` | The page. Production DOM with every remote URL repointed at `assets/`. |
| `assets/` | Fonts, images, video, JS and CSS (~150 MB). |
| `serve.js` | Minimal static server (correct MIME types for `.avif` / `.webm` / `.woff2`). |
| `CLONE-PROMPT.md` | Full technical teardown — animation timings, scroll ranges, interaction specs. |
| `reference/` | Original source files plus frame-by-frame capture evidence. |

## How it was verified

Measured against the live site in a real browser rather than eyeballed. Both pages report identical values:

| | Live | This replica |
|---|---|---|
| Logo FLIP scroll range | `0 → 1188px` | `0 → 1188px` |
| Section heights | 2700 / 3119 / 3600 / 1239 / 1233 / 1056 / 846 | identical |
| Total page height | `13857px` | `13857px` |
| Intro timeline | `3.53s`, `heroReveal` at `1.4` | identical |
| Swiper config | 8 slides, `spaceBetween: 14`, `speed: 500`, no autoplay | identical |
| Console errors | 0 | 0 |

Interactions confirmed working: work-card group-hover dimming, hover-triggered foreground video, FAQ accordion, and swiper drag with the custom `DRAG` cursor.

Frame captures live in `reference/intro-frames/` (intro scrubbed by pausing GSAP's own clock) and `reference/scroll-frames/` (scroll positions set deterministically with `ScrollTrigger.update()`).

**Untested:** the Safari `.mov` video branch. Verification ran in Chromium, which always takes the `.webm` path.

## Attribution & licensing

All design, copy, imagery and media belong to Nenad Popadic and the respective client brands (1910.ai, SemiconBio, Happy Ring, Puck, Alosant, Omicron, PSSLTD, Lilipad, GetRay). Testimonials name real people.

`assets/fonts/` contains **commercially licensed typefaces** — PP Neue Montreal and TR 3 A — included only so the replica renders locally. They are not licensed for redistribution.

Nothing here is original work by the repository owner, and none of it is cleared for republication. Keep this repository private.
