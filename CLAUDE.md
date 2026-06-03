# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project shape

Single-file prototype: everything lives in `index.html` (vanilla JS, inline `<style>`, no build step, no dependencies, no package.json). The empty `타로점을 보는 앱/` directory is a leftover folder name and can be ignored.

Run by opening `index.html` directly in a browser, or serve statically (`python -m http.server 8000`). There is no test suite, lint config, or build command.

Deploy = push to `main`; GitHub Pages can serve the root directly.

## Architecture inside `index.html`

Three-screen SPA driven by toggling `.active` on `<section class="screen">` elements (`#screen-intro`, `#screen-pick`, `#screen-result`). All screen swaps go through `showScreen(name)` — do not toggle `display`/classes elsewhere.

State is a single module-level `state` object:
- `state.question` — user's question text
- `state.shuffledDeck` — `TAROT_DECK` shuffled per session
- `state.picked` — array of indices into `shuffledDeck`, max length 3
- `state.results` — `[{ card, isReversed }]` produced at result-screen entry

`TAROT_DECK` is the source of truth for card data (`num`, `name`, `nameEn`, `symbol`, `upright`, `reversed`). When adding cards, keep the same shape — `buildCardEl` and the interpret renderer both read these fields directly.

Card visuals are **CSS-only** (no `<img>` tags). The card "face" is a unicode symbol + text inside a gradient div, and reversed orientation is rendered by rotating `.card-front-content` 180°. Do not introduce image files for cards — the original brief explicitly chose CSS to avoid broken-image errors.

Card flip uses 3D CSS: `.card-inner` with `transform-style: preserve-3d` and `.card.flipped` toggling `rotateY(180deg)`. Both faces use `backface-visibility: hidden`.

## Invariants to preserve

- **Pick guard**: `onPickCard` must check both `state.picked.includes(idx)` and `state.picked.length >= 3` before mutating. Don't rely on the `.picked` CSS class alone.
- **Reset completeness**: `#btn-reset` handler must clear every `state.*` field, the three dynamic containers (`#deck-area`, `#result-cards`, `#interpret-list`), and the `#question-input` value. Adding new state requires extending this handler.
- **Mobile-first layout**: the app is locked to `max-width: 480px` and uses `aspect-ratio` on cards so they never overflow. Grid drops to 3 columns under 360px. Avoid fixed pixel card sizes.
- **Korean UI copy**: all user-facing strings are Korean; keep that consistent when editing.
