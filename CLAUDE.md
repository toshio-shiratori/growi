# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Infrastructure and static assets for services operated by シンプルアップ工房 on the `sastd.com` domain. Contains:

1. **Caddy reverse proxy config** (`Caddyfile`) — routes traffic for `pbank-api.sastd.com` (P-BANK API) and `growi.sastd.com` (GROWI wiki). Includes security headers, HSTS, and JSON access logging.
2. **GROWI wiki stack** (`growi/`) — Docker Compose setup running GROWI 7, MongoDB 8, and Elasticsearch 9 (with kuromoji + ICU plugins for Japanese text analysis). Site URL: `https://growi.sastd.com`.

## Development

- **GROWI stack**: `docker compose -f growi/docker-compose.yml up -d`. Requires the external `pbank-network` Docker network to exist.
- **Caddy**: Deployed as part of the server's Caddy instance; the `Caddyfile` is not run locally.

## Notes

- The GROWI Elasticsearch image installs `analysis-kuromoji` and `analysis-icu` for Japanese full-text search.
- The Caddy config blocks Swagger UI in production (`/swagger-ui/*` returns 404).
