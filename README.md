# AI News Hub

A single-page news aggregator that surfaces the top 5 AI headlines of the moment as plain-English, jargon-free summaries. It's built for people who need to stay AI-literate but don't have time to read primary sources — a marketing manager, a founder, anyone who wants five minutes of signal instead of an hour of noise. The whole product is a static HTML/CSS/JS page: no backend, no build step, no dependencies.

**[Live demo →](index.html)** (open directly in a browser, or serve locally — see below)

---

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

Deploys as-is to any static host — Vercel, Netlify, GitHub Pages — with zero configuration.

## What's implemented (MVP / Phase 1)

- Card grid of 5 curated articles: headline, source, plain-English summary, tags, and a link to the original piece
- Category filtering (All / Anthropic & Claude / Research & Breakthroughs), driven by client-side array filtering
- Manual refresh with a loading state and a "last updated" timestamp
- Mobile-first responsive layout (single column under 768px, auto-fit grid above it)
- Graceful image degradation: article images fall back to an emoji if the source image fails to load

## Browser support

Latest two versions of Chrome, Firefox, Safari, and Edge. No polyfills — the JS used (template literals, arrow functions, `fetch`-shaped async patterns, CSS Grid) is baseline-supported across all four without transpilation.

---

## Future phases

This build covers Phase 1 only. The planned phases beyond it:

### Phase 2: Growth
- Email newsletter integration (daily/weekly digest)
- Search functionality
- Advanced filtering (by date, company, topic)
- User preferences (saved/favorited articles)
- Homepage widget version

### Phase 3: Scale
- Automated scraping / news-API integration, replacing manual curation
- Smart recommendations
- Premium tier launch
- Browser extension
- Mobile app (PWA first)

### Phase 4: AI-Powered
- Personalized summaries via Claude
- Audio summaries for commuters
- Slack integration
- Corporate dashboard version
