---
name: web-search
description: "Search and scrape the web using local infrastructure: SearXNG + Crawl4AI, with browser tools as fallback."
version: 1.0.0
metadata:
  tags: [web-search, searxng, crawl4ai, scraping]
---

# Web Search via SearXNG + Crawl4AI

Use this skill whenever web search or URL scraping is needed. Priority order:

1. **SearXNG** (local) — discover URLs via meta-search
2. **Crawl4AI** (local) — extract clean markdown/HTML from URLs
3. **Hermes browser tools** — fallback when local tools fail

## Infrastructure

| Service | Localhost Port | Role |
|---------|---------------|------|
| SearXNG | 8081 | Meta-search (DuckDuckGo, Google, Brave, Startpage) |
| Crawl4AI | 11235 | Web scraping → markdown / HTML extraction |

## Step 1 — Search (SearXNG)

```bash
curl -s 'http://localhost:8081/search?q=<query>&format=json'
```

Response fields per result: `title`, `url`, `content` (snippet), `engine`, `score`, `publishedDate`.

- Skip low-score results (< 0.3).
- If empty, rephrase or fall back to browser tools.

## Step 2 — Scrape (Crawl4AI)

Markdown extraction (LLM-friendly):
```bash
curl -s -X POST http://localhost:11235/md \
  -H 'Content-Type: application/json' \
  -d '{"url": "<URL>"}'
```

Raw HTML:
```bash
curl -s -X POST http://localhost:11235/html \
  -H 'Content-Type: application/json' \
  -d '{"url": "<URL>"}'
```

Both accept JSON body with `url` key. Response: `success`, `markdown`/`html`.

## Browser Fallback

If services are down, use Hermes browser tools directly:
1. `browser_navigate()` → search engine or target URL
2. `browser_snapshot()` → extract structured text
3. `browser_scroll()` → reveal more content

## Health Checks

```bash
curl -s 'http://localhost:8081/search?q=test&format=json' | head -3
curl -s -X POST http://localhost:11235/md -H 'Content-Type: application/json' -d '{"url":"https://example.com"}' | head -3
docker ps --format '{{.Names}} {{.Status}}' | grep -iE 'searxng|crawl4ai'
```

## Error Handling

- **SearXNG empty**: rephrase query; check `curl http://localhost:8081/` for HTML response
- **Crawl4AI "Method Not Allowed"**: must use POST, not GET
- **Both down**: fall back to browser tools
