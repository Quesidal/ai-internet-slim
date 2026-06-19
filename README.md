# Internet Layer (Slim) — SearXNG + Crawl4AI

Minimal self-hosted web search + scraping stack for AI agents. Two containers, no API keys, no rate limits.

## Services

| Service | Port | Purpose |
|---------|------|---------|
| **SearXNG** | 8081 | Meta-search (DuckDuckGo, Google, Brave, Startpage) |
| **Crawl4AI** | 11235 | Web scraping → clean markdown for LLMs |

## Requirements

- Docker + Docker Compose (v2.20+)
- ~6GB disk space for images (first run)
- ~2GB RAM minimum

## Quick Start

```bash
cd internet-layer-slim

# 1. Create .env with a secret key
cp .env.example .env
# Edit .env — replace SEARXNG_SECRET with: openssl rand -hex 32

# 2. Update SearXNG settings.yml
# Replace "CHANGE-THIS-SECRET-KEY" in volumes/searxng-settings/settings.yml
# with the same value from .env

# 3. Start
docker compose up -d

# 4. Verify
docker compose ps
```

## API Endpoints

### Search (SearXNG)
```bash
curl 'http://localhost:8081/search?q=your+query&format=json'
```

Returns JSON with `results` array. Each result: `title`, `url`, `content` (snippet), `engine`, `score`.

### Scrape (Crawl4AI)
```bash
# Markdown (LLM-friendly)
curl -X POST http://localhost:11235/md \
  -H 'Content-Type: application/json' \
  -d '{"url": "https://example.com"}'

# Raw HTML
curl -X POST http://localhost:11235/html \
  -H 'Content-Type: application/json' \
  -d '{"url": "https://example.com"}'
```

## Hermes Integration

Drop the skill into your Hermes skills directory:
```bash
mkdir -p ~/.hermes/skills/web-search
cp web-search/SKILL.md ~/.hermes/skills/web-search/
```

Teaches Hermes to use SearXNG for search + Crawl4AI for scraping, with browser tools as fallback.

## Resource Usage

| Service | Memory Limit | CPU |
|---------|-------------|-----|
| SearXNG | 512MB | Low |
| Crawl4AI | 4GB (1G reserved) | Medium |

## Customizing Ports

Edit `.env`:
```
SEARXNG_PORT=8081
CRAWL4AI_PORT=11235
```

## License

MIT. Made by Alex (@aistation).
