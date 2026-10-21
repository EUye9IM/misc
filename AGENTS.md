# misc — Agent Guide

This repo contains infrastructure configurations and reusable Hermes skills.

---

## firecrawl/ — Self-Hosted Firecrawl

### Architecture

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

### Proxy Configuration

Two proxy channels:

| Variable | Protocol | Used By |
|----------|----------|---------|
| `ALL_PROXY` | `http://...` | Node.js `undici` fetch, curl, Python (SearXNG) |
| `PROXY_SERVER` | `socks5://...` | Playwright / Chromium browser engine |

Chromium requires SOCKS5 for HTTPS pages through a proxy; Node.js and Python tools prefer HTTP proxy.

- `no_proxy` — bypass list (default: `localhost,127.0.0.1,.local,192.168.0.0/16,172.16.0.0/12`)

### Setup

```bash
cp firecrawl/.env.example firecrawl/.env
# Fill in PROXY_SERVER, ALL_PROXY, POSTGRES_PASSWORD, etc.
docker compose -f firecrawl/docker-compose.yaml up -d
```

### Conventions

- `network_mode: host` — no port mapping
- `USE_DB_AUTHENTICATION=false` — no API key needed locally
- SearXNG JSON API at `http://localhost:8080/search?format=json&q=...`
- Playwright browser cache on `tmpfs` (1GB)
- `.env` is gitignored; use `.env.example` as template
