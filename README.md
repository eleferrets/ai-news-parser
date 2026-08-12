# AI News Hub

A single-page news aggregator that surfaces the top 5 AI headlines of the moment as plain-English, jargon-free summaries. It's built for people who need to stay AI-literate but don't have time to read primary sources — a marketing manager, a founder, anyone who wants five minutes of signal instead of an hour of noise. The whole product is a static HTML/CSS/JS page: no backend, no build step, no dependencies. See [`docs/PRD.md`](docs/PRD.md) for the full product spec this implements (Phase 1 / MVP scope).

**[Live demo →](index.html)** (open directly in a browser, or serve locally — see below)

---

## Why this repo exists

This is a small project, on purpose. It's sized to be read end-to-end in one sitting, which makes it a good vehicle for talking through *how* to make software decisions, not just what the finished feature looks like. The section below is written for that conversation — it's the "why," not just the "what."

## Running it

There's nothing to install.

```bash
# Option 1: just open it
open index.html          # macOS
xdg-open index.html      # Linux

# Option 2: serve it (recommended, avoids file:// quirks with some browsers)
python3 -m http.server 8000
# then visit http://localhost:8000
```

Deploys as-is to any static host — Vercel, Netlify, GitHub Pages — with zero configuration, per the PRD's hosting requirement.

## What's implemented (MVP / Phase 1)

- Card grid of 5 curated articles: headline, source, plain-English summary, tags, and a link to the original piece
- Category filtering (All / Anthropic & Claude / Research & Breakthroughs), driven by client-side array filtering
- Manual refresh with a loading state and a "last updated" timestamp
- Mobile-first responsive layout (single column under 768px, auto-fit grid above it)
- Graceful image degradation: article images fall back to an emoji if the source image fails to load

---

## Technical decisions and tradeoffs

### 1. Vanilla JS over a framework
**Decision:** No React/Vue/build tooling — plain HTML, CSS, and DOM APIs in one file.
**Why:** At 5 articles and 3 filter states, a framework buys nothing — no component tree deep enough to need reconciliation, no state complex enough to need a store. A build step would add a compile target, a `node_modules`, and a deploy pipeline for a page that's otherwise just static assets.
**Tradeoff:** This doesn't scale gracefully. The moment the PRD's Phase 2 features land (search, saved favorites, personalization), the hand-rolled `innerHTML` re-render approach starts fighting you — no component boundaries, no diffing, manual event re-wiring after every render. I'd treat "introduce a framework" as a deliberate, scoped migration once state stops being "an array and a filter string," not something to preempt now.

### 2. Full re-render via `innerHTML` instead of incremental DOM updates
**Decision:** `renderArticles()` rebuilds the whole grid from a template string on every filter/refresh, rather than diffing and patching individual cards.
**Why:** Simpler to write and reason about, and at N=5 cards the reflow cost is unmeasurable.
**Tradeoff:** This is a decision I would *not* make the same way past a few dozen items, or if cards had internal state (e.g., an expand/collapse toggle) that a full re-render would blow away. It's also why there's no animation on filter transitions in this build — you'd need to keep DOM nodes alive across renders to animate them.

### 3. Static, embedded data instead of a live feed
**Decision:** Articles live in a JS array in the page, matching the PRD's `6.2 Data Structure`. Refresh currently simulates a fetch with `setTimeout`.
**Why:** The PRD explicitly scopes MVP to manual weekly curation (`6.3`) — automated scraping and news-API integration are Phase 2/3. Building the scraper before validating that people want the curated digest would be solving a problem nobody's confirmed exists yet.
**Tradeoff:** Freshness is bounded by however often someone edits the array. The `fetchArticles()` function is deliberately shaped like it's awaiting a real network call so that swapping the `setTimeout` for a `fetch()` against a JSON endpoint (or a news API) is a contained, one-function change rather than a rewrite — that's the seam Phase 2 hooks into.

### 4. Client-side filtering
**Decision:** Category filters run `articles.filter()` in the browser against the already-loaded dataset.
**Why:** With 5 items there's no latency or payload-size argument for server-side filtering — it would just add a round trip.
**Tradeoff:** This inverts once the catalog is large enough that shipping the full dataset to the client is wasteful, or once filtering needs to combine with pagination/search across a dataset the client shouldn't hold in memory. That's a server-side-query problem, not a client one.

### 5. Trusted-input assumption in the render path
**Decision:** Article fields (`title`, `summary`, `source`, `tags`) are interpolated directly into template literals and written via `innerHTML`, without escaping.
**Why:** This is safe *today* specifically because every field is developer-curated — nothing here is user- or API-sourced.
**Tradeoff:** This is the assumption most likely to bite someone later. The instant this data starts coming from an API response, a scraper, or any external submission (Phase 2/3, or the newsletter/premium features), this becomes a stored-XSS vector — an article title or summary could carry a `<script>` or event handler straight into the DOM. The fix at that point is either an escaping helper on interpolation or switching to `textContent`/DOM node construction for anything not fully trusted. I'd flag this as the first thing to fix before wiring in any external data source, not something to defer.

### 6. `target="_blank"` links use `rel="noopener noreferrer"`
**Decision:** Outbound article links open in a new tab with `rel="noopener noreferrer"` set.
**Why:** `target="_blank"` without it is a known reverse-tabnabbing vector — the opened page gets a `window.opener` reference back into this page and can navigate it. Trivial to close, easy to forget, so it's set explicitly here rather than left as a follow-up.

### 7. Resilient but unverified image loading
**Decision:** Article images are hotlinked from an external host, with an `onerror` handler that swaps in an emoji placeholder on load failure.
**Why:** No image hosting/CDN needed for an MVP — and the fallback means a dead or slow-loading image degrades the experience instead of breaking it.
**Tradeoff:** Hotlinking third-party images is inherently fragile — no control over availability, no cache-busting, and it depends on the third party's hotlink policy. It also means image correctness isn't verified at build time; a malformed URL just silently falls back to the emoji rather than failing loudly. That's the right failure mode for a reader-facing product (never show a broken-image icon), but it's the kind of thing a linter or a CI content check should catch before merge in a team setting — right now nothing validates the curated data at all.

### 8. No client-side persistence (yet)
**Decision:** Filter state resets on reload; nothing is written to `localStorage`.
**Why:** The PRD marks filter persistence as optional for MVP (`4.2`). Adding storage now would be state to manage for a benefit nobody asked to validate first.
**Tradeoff:** Straightforward to add later (`localStorage.setItem` on filter change, read on load) — deferred because it's pure upside with no design risk, so it's not worth doing before something that actually needs validating.

### 9. Accessibility as a constraint, not an afterthought
**Decision:** Filter and refresh controls are real `<button>` elements (native keyboard/focus support for free), images carry `alt` text, and the color palette was chosen against the PRD's WCAG AA requirement (`4.4`).
**Gap:** No explicit focus-visible styling beyond the browser default, and contrast hasn't been run through an automated checker (e.g. axe/Lighthouse) — it's eyeballed against the AA guideline, not verified. That verification step is the honest next item before calling this launch-ready, per the PRD's own acceptance criteria (`15`).

---

## Roadmap

This build is Phase 1 of the PRD's four-phase plan. Phase 2 (search, saved favorites, newsletter), Phase 3 (real scraping/news-API integration, premium tier), and Phase 4 (personalized/AI-generated summaries) are documented in [`docs/PRD.md § 11`](docs/PRD.md#11-roadmap) and intentionally out of scope here — each one changes an architectural assumption above (static data, no persistence, no backend) rather than just adding a feature, which is why they're phased instead of bundled in.

## Browser support

Latest two versions of Chrome, Firefox, Safari, and Edge, per the PRD (`6.1`). No polyfills — the JS used (template literals, arrow functions, `fetch`-shaped async patterns, CSS Grid) is baseline-supported across all four without transpilation.
