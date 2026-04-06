# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Infrastructure and static assets for services operated by シンプルアップ工房 on the `sastd.com` domain. Contains:

1. **Caddy reverse proxy config** (`Caddyfile`) — routes traffic for `pbank-api.sastd.com` and `growi.sastd.com`. Includes security headers, HSTS, and JSON access logging.
2. **GROWI wiki stack** (`growi/`) — Docker Compose setup running GROWI 7, MongoDB 8, and Elasticsearch 9. Site URL: `https://growi.sastd.com`.

## Architecture

### Caddy routing (`Caddyfile`)

- **pbank-api.sastd.com**: Path-based routing to `pbank-api:8080` — `/api/*`, `/admin/*`, `/login/oauth2/*`, `/actuator/health` are proxied; `/swagger-ui/*` returns 404; all other paths return 404.
- **growi.sastd.com**: Full reverse proxy to `growi-app:3000`. TLS via ACME with ALPN challenge disabled.

### GROWI stack (`growi/docker-compose.yml`)

Three services on an internal `growi-network`:

| Service | Image | Container | Purpose |
|---|---|---|---|
| `growi` | `growilabs/growi:7` | `growi-app` | Wiki app (port 3000) |
| `mongo` | `mongo:8.0` | `growi-mongo` | Document store |
| `elasticsearch` | Custom build (`growi/elasticsearch/v9/`) | `growi-es` | Full-text search |

The `growi` service also joins the external `pbank-network` so Caddy can reach it.

### Elasticsearch custom image

`growi/elasticsearch/v9/Dockerfile` extends `elasticsearch:9.0.3` with `analysis-kuromoji` and `analysis-icu` plugins for Japanese full-text search. Security (`xpack.security`) is disabled in the config.

## Commands

```bash
# Create the external network (one-time, required before first start)
docker network create pbank-network

# Start GROWI stack
docker compose -f growi/docker-compose.yml up -d

# Rebuild Elasticsearch image after Dockerfile changes
docker compose -f growi/docker-compose.yml build elasticsearch

# View logs
docker compose -f growi/docker-compose.yml logs -f [growi|mongo|elasticsearch]

# Stop stack
docker compose -f growi/docker-compose.yml down
```

## Notes

- Caddy is deployed as part of the server's Caddy instance; the `Caddyfile` is not run locally.
- `GROWI_PASSWORD_SEED` is set in `.env` (git-ignored) and passed to the GROWI container.
- Elasticsearch JVM heap is set to 256 MB (`ES_JAVA_OPTS=-Xms256m -Xmx256m`).
