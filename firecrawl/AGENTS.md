# Firecrawl Self-Hosted — Agent Guide

## Project Overview

Self-hosted Firecrawl instance with SearXNG metasearch, running via Docker Compose with host networking. Provides web scraping and search API endpoints.

## Architecture

| Service | Port | Description |
|---------|------|-------------|
| Firecrawl API | `3002` | Scrape & search API |
| Playwright | `3000` | Headless browser for JS-rendered pages |
| SearXNG | `8080` | Privacy-respecting metasearch engine |
| Redis | `6379` | Job queue & caching |
| RabbitMQ | `5672` | Message broker |
| PostgreSQL | `5432` | Job metadata store |

All containers use `network_mode: host` — no port mapping; services communicate via localhost.

### Data Flow

API workers receive scrape/search jobs → enqueue via RabbitMQ → workers fetch via Playwright or SearXNG → results cached in Redis → metadata persisted to PostgreSQL.

## Proxy Configuration

Two proxy channels are configured:

| Variable | Protocol | Used By |
|----------|----------|---------|
| `ALL_PROXY` | `http://...` | Node.js `undici` fetch, curl, Python (SearXNG) |
| `PROXY_SERVER` | `socks5://...` | Playwright / Chromium browser engine |

Chromium requires SOCKS5 for HTTPS pages through a proxy; Node.js and Python tools prefer HTTP proxy. This split is why both variables exist.

- `no_proxy` — comma-separated bypass list for both channels (default: `localhost,127.0.0.1,.local,192.168.0.0/16,172.16.0.0/12`)

Wired into:
- **Firecrawl API workers** — `ALL_PROXY` via Node.js `undici`
- **Playwright** — `PROXY_SERVER` for Chromium + `ALL_PROXY` for other tools
- **SearXNG** — `ALL_PROXY` via Python `requests`/`urllib3`

## Setup

```bash
cp .env.example .env
# Fill in PROXY_SERVER, ALL_PROXY, POSTGRES_PASSWORD, BULL_AUTH_KEY, searxng/settings.yml secret_key
docker compose up -d
```

### Secrets to replace

| Variable | Default | Notes |
|----------|---------|-------|
| `POSTGRES_PASSWORD` | `change-me` | DB password |
| `BULL_AUTH_KEY` | `change-me` | Queue dashboard auth |
| `searxng/settings.yml:secret_key` | `change-me-to-a-random-secret` | Generate via `openssl rand -base64 32` |
| `PROXY_SERVER` | (none) | SOCKS5 proxy address |
| `ALL_PROXY` | (none) | HTTP proxy address |

## API Usage

```bash
# Search
curl -s http://localhost:3002/v1/search -H 'Content-Type: application/json' \
  -d '{"query":"your search terms"}'

# Scrape
curl -s http://localhost:3002/v1/scrape -H 'Content-Type: application/json' \
  -d '{"url":"https://example.com","timeout":30000}'
```

## Conventions

- **`network_mode: host`** — all containers share the host network; no port mapping in compose
- **`USE_DB_AUTHENTICATION=false`** — API key auth disabled for local use
- **SearXNG JSON API** — `http://localhost:8080/search?format=json&q=...`
- **Playwright tmpfs** — browser cache mounted as `tmpfs` with 1GB limit
- **Env file** — `.env` must be created from `.env.example`; proxy vars must be set for external access
- **Secrets** — do not commit `.env` to version control
