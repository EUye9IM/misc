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

Two proxy channels are configured for different components:

| Variable | Protocol | Used By |
|----------|----------|---------|
| `ALL_PROXY` | `http://...` | Node.js `undici` fetch, curl, Python tools (SearXNG) |
| `PROXY_SERVER` | `socks5://...` | Playwright / Chromium browser engine |

This split is necessary because Chromium requires SOCKS5 for HTTPS pages through a proxy, while Node.js and Python tools prefer HTTP proxy.

- `no_proxy` — comma-separated bypass list for both channels

These are wired into:
- **Firecrawl API workers** — via `ALL_PROXY` (Node.js `undici` HTTP client)
- **Playwright service** — via `PROXY_SERVER` (Chromium `browser.newContext({ proxy })`) + `ALL_PROXY` for other tools
- **SearXNG** — via `ALL_PROXY` (Python `requests`/`urllib3`)

Default bypass: `localhost,127.0.0.1,.local,192.168.0.0/16,172.16.0.0/12`

## Secrets to Replace

| Variable | Default | Notes |
|----------|---------|-------|
| `POSTGRES_PASSWORD` | `change-me` | DB password |
| `BULL_AUTH_KEY` | `change-me` | Queue dashboard auth |
| `searxng/settings.yml:secret_key` | `change-me-to-a-random-secret` | Cookie signing key, generate via `openssl rand -base64 32` |
| `PROXY_SERVER` | `socks5://...` | Your SOCKS5 proxy address |
| `ALL_PROXY` | `http://...` | Your HTTP proxy address |

## Notes

- Uses `network_mode: host` — containers share host network (no port mapping needed)
- `USE_DB_AUTHENTICATION=false` — API key auth disabled for local use
- SearXNG exposes JSON API at `http://localhost:8080/search?format=json&q=...`
- The Playwright service uses `tmpfs` for cache (1GB limit)
