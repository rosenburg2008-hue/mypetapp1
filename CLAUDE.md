# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`멍냥마당` (Meongnyang Madang) — a Korean-language **planning proposal / clickable 시안 (design mockup)** for a neighborhood-based pet-care portal serving dog and cat owners equally. Authored by 로젠버그 박사. It doubles as a market-research report (시장 전수조사) and a development pitch.

It is **not a production app**. Interactions are simulated, all data is hardcoded, and nothing persists. Cards are footnoted `시안용 예시, 출시 전 수의사 검수 예정` for this reason — keep that framing when adding content.

## Commands

There is no build, test, lint, or dependency step. The entire site is one static file.

```bash
open index.html                 # preview (macOS)
python3 -m http.server 8000     # or serve, then visit localhost:8000
```

Verify changes by loading the page in a browser and exercising the species toggle, search, and filters. Check the browser console for errors — a syntax error anywhere in the single `<script>` block silently kills every interaction on the page, since it is one IIFE.

## File layout

`index.html` is self-contained — no bundler, no framework, no external JS. The only network dependency is Google Fonts (Gowun Dodum for body, Jua for headings). Three blocks:

| Lines | Contents |
|---|---|
| 11–206 | `<style>` — all CSS, custom properties on `:root` |
| 208–689 | Markup — SVG sprite, sticky header, `<main>` sections, footer |
| 690–888 | `<script>` — one IIFE holding all data and all behavior |

## Architecture

### Species filter is the cross-cutting concern

`state.sp` (`'all' | 'dog' | 'cat'`) is set by the header segmented control (`[data-sp-btn]`) and drives nearly everything. `setSpecies()` re-runs `renderInfo()`, `renderFeed()`, `renderEvents()`, and `panels()`, and syncs the compose dropdown.

`spOk(sp)` is the filter predicate: a record passes when the filter is `'all'`, **or the record itself is `'all'`**, or they match. Records tagged `sp:'all'` are therefore always visible — use that tag for content relevant to both species.

The walk-matcher and cat-group panels are *not* filtered by `spOk`. Instead `panels()` hides the whole panel via the `hidden` attribute, because a dog walk matcher makes no sense for cat owners. This asymmetry is intentional and stated as a design principle in the 개발 제안 section ("고양이는 고양이답게").

### Content lives in JS arrays, not markup

Five arrays at the top of the IIFE are the editable content source. To add or change a card, post, event, etc., edit the array — not the HTML.

- `INFO` — info cards (`sp`, `cat`, `t` title, `b` body). Category filter buttons are **derived** from the distinct `cat` values, so a new category string automatically creates a new filter chip.
- `FEED` — 동네 마당 posts
- `DOGS` — walk-mate candidates (filtered by the `mSize` / `mTime` selects, read directly from the DOM rather than from `state`)
- `CATS` — cat-owner group activities
- `EVENTS` — event calendar rows

The 시장 전수조사, 개발 제안, and 단계별 개발 계획 sections are hand-written static HTML, **not** data-driven. Edit their markup directly.

### Rendering and event handling

Every renderer rebuilds its container with `innerHTML` from string concatenation, wholesale, on every change. There is no diffing and no persistence — a reload resets all state.

Two rules govern this code:

1. **Interpolate user- or content-derived strings through `esc()`.** Only the internal `sp` enum is written raw (into `data-sp` and `#i-` sprite refs).
2. **Resolve indices against the master array, not the filtered list.** Renderers filter first, then compute `idx` via `ARRAY.indexOf(item)` and emit it as `data-i` / `data-d` / `data-c` / `data-e`. Delegated click handlers on the container read that attribute back. Using the filtered list's index here would mutate the wrong record whenever a filter is active.

Toggle state is stored as an ad-hoc flag on the data object itself (`p.liked`, `d.sent`, `c.on`, `v.on`), flipped in the handler, then re-rendered.

### `aria-pressed` is the state, including visually

Selected/active styling is keyed entirely off `[aria-pressed="true"]` in CSS (`.seg button`, `.cats button`, `.sbtn`, `.like`). There are no `.active` or `.selected` classes. Set `aria-pressed` and the visual state follows — do not add a parallel class.

### Species color coding

Applied consistently across cards, chips, panels, gap callouts, and roadmap phases:

- **dog** → `--ball` yellow (`#F4C542`), soft `--ballSoft`, text `--ballInk`
- **cat** → `--yarn` purple (`#9A86D6`), soft `--yarnSoft`, text `--yarnInk`
- **all / both** → `--grass` green (`#3F8A5F`)

Reuse these tokens rather than introducing new colors. `.sec.alt` gives a section the pale `--leaf` background; `.sec.dark` inverts to the dark `--ink` ground and needs explicit light text overrides.

Dog and cat faces are SVG `<symbol>`s (`#i-dog`, `#i-cat`) defined once near the top of `<body>` and drawn with `<use href="#i-...">`. Their fills are hardcoded hex, not variables.

## Conventions to preserve

- **Korean copy throughout**, in plain polite style (`합니다`/`하세요`). `word-break:keep-all` is set globally so Korean phrases wrap at word boundaries — do not override it.
- **Accessibility is already wired in** and should stay that way: skip link, `aria-labelledby` on every section, `aria-live` on dynamically re-rendered containers, `aria-label` on bare controls, a visible `:focus-visible` ring, and a `prefers-reduced-motion` block that disables the hero draw-on animation and smooth scrolling.
- **ES5-style JS** — `var`, `function`, no arrow functions or template literals in the existing code. Match it.
- **Responsive collapse at 900px**: every multi-column grid drops to one column and the header wraps its nav to a third row (which is why `scroll-padding-top` increases to 120px there). Wide tables scroll horizontally inside `.tscroll` instead of collapsing.
- **Cite claims.** Figures in the research tables trace to the numbered `<ol>` in the `#sources` footer. Do not add statistics without a corresponding source entry, and preserve the existing practice of showing conflicting estimates from different institutions rather than silently picking one. Unverified numbers (e.g. app user counts) are deliberately omitted.
