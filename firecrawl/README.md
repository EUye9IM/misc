# Firecrawl Self-Hosted

Self-hosted [Firecrawl](https://github.com/nicehash/ergo-agent) instance with SearXNG metasearch, running via Docker Compose with host networking.

## Services

| Service | Port | Description |
|---------|------|-------------|
| Firecrawl API | `3002` | Web scraping & search API |
| Playwright | `3000` | Headless browser for JS-rendered pages |
| SearXNG | `8080` | Privacy-respecting metasearch engine |
| Redis | `6379` | Job queue & caching |
| RabbitMQ | `5672` | Message broker |
| PostgreSQL | `5432` | Job metadata store |

## Quick Start

```bash
# Copy and edit env
cp .env.example .env
# Fill in your proxy and secrets
# Then start
docker compose up -d
```

## API Usage

```bash
# Search
curl -s http://localhost:3002/v1/search \
  -H 'Content-Type: application/json' \
  -d '{"query":"your search terms"}'

# Scrape a page
curl -s http://localhost:3002/v1/scrape \
  -H 'Content-Type: application/json' \
  -d '{"url":"https://example.com","timeout":30000}'
```

## Proxy Configuration

The proxy is configured via environment variables:

- `ALL_PROXY` — proxy for all HTTP/HTTPS traffic (e.g. `http://proxy:8080`)
- `no_proxy` — comma-separated bypass list (`localhost,127.0.0.1,.local,10.0.0.0/8,...`)

These are wired into:
- Firecrawl API workers (via `undici` — Node.js HTTP client)
- Playwright service (via `browser.newContext({ proxy })` + Chromium env)
- SearXNG (via `outgoing.proxies` removed in favor of `ALL_PROXY` env var)

## Secrets to Replace

| Variable | Default | Notes |
|----------|---------|-------|
| `POSTGRES_PASSWORD` | `firecrawl_password` | DB password |
| `BULL_AUTH_KEY` | `firecrawl-bull-key` | Queue dashboard auth |
| `searxng/settings.yml:secret_key` | `change-me-to-a-random-secret` | Cookie signing key, generate via `openssl rand -base64 32` |

## Notes

- Uses `network_mode: host` — containers share host network (no port mapping needed)
- `USE_DB_AUTHENTICATION=false` — API key auth disabled for local use
- SearXNG exposes JSON API at `http://localhost:8080/search?format=json&q=...`
- The Playwright service uses `tmpfs` for cache (1GB limit)
