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

---

## skills/3gpp-skill/ — 3GPP Telecom Expert Skill

A comprehensive 3GPP telecommunications skill for Hermes (forked from `lugasia/3gpp-skill`), covering GSM through 6G.

### Structure

| File | Purpose |
|------|---------|
| `SKILL.md` | Main skill instructions — knowledge domains, response patterns |
| `references/releases.md` | Release-by-release feature breakdown (Phase 1 → Rel-21) |
| `references/phy-layer.md` | PHY layer deep-dive (synchronization signals, channels, RACH) |
| `references/working-groups.md` | RAN/SA/CT WG structure with spec ownership |
| `references/download.md` | How to download 3GPP specs from ftp.3gpp.org |
| `references/reading-docx.md` | Methodology for reading long spec .docx files |

### Response Patterns

When answering 3GPP questions:

1. Refer to the relevant references (releases, PHY, WGs)
2. For detailed protocol/procedure questions, **download the spec and read it directly** — see `references/download.md` + `references/reading-docx.md`
3. Ground answers in actual TS spec text, not training data
4. Always cite the spec number and section

### Spec Download Workflow

1. List versions: `curl -s "ftp://ftp.3gpp.org/Specs/archive/{series}_series/{spec}/"`
2. Pick latest (highest version code letter)
3. Download: `curl -LO "ftp://ftp.3gpp.org/Specs/archive/{path}/{file}.zip"`
4. Unzip: `unzip -o {file}.zip`
5. Read .docx with `python-docx` or `read_file`

### Version Codes

| Letter | Release |
|--------|---------|
| f | Rel-15 |
| g | Rel-16 |
| h | Rel-17 |
| i | Rel-18 |
| j | Rel-19 |
| k | Rel-20 |
