# CETECH ERPNext — Hetzner + Dokploy

**Status: CONFIGURATION PREPARED — NOT DEPLOYED.**

**Confirmed intended site hostname:** `erpnext.benuel.dedyn.io` (DNS and HTTPS routing must be verified before deployment).

This repository holds the Docker Compose configuration for CETECH's ERPNext stack. Dokploy already runs on the Hetzner server and provides external routing and HTTPS via Traefik. The official ERPNext frontend container uses Nginx internally.

## Architecture

- Docker Compose in Dokploy (**not** Docker Stack).
- Official ERPNext image with a pinned release tag; initial candidate: **v16.50.0**.
- Services: MariaDB, Redis cache, Redis queue, configurator, backend, frontend, WebSocket, scheduler, and short/long workers.
- No host-published ports; map Dokploy Domains to **frontend, internal port 8080**.
- Persistent Docker named volumes for sites, MariaDB and Redis queue data.
- **One-time** manual ERPNext site initialization, never on every redeploy.

## Repository contents

- [docker-compose.yml](./docker-compose.yml) — Dokploy-ready Compose configuration.
- [.env.example](./.env.example) — non-secret variables example. Copy values into Dokploy Environment, never commit a real .env.
- [.gitignore](./.gitignore) — blocks local secrets, backups and site data.
- [Deployment runbook](./docs/DEPLOYMENT.md) — preflight, Dokploy connection, domain, initialization and acceptance testing.
- [Operations runbook](./docs/OPERATIONS.md) — backup, restore readiness, safe updates and credential handling.

## Deployment checklist

1. Inspect the Hetzner host's CPU, RAM, disk, existing apps, network and volumes.
2. In Dokploy create/select a Compose service using repository **Ben-001-sys/cetech-erp**, branch **main**, path **./docker-compose.yml**, **Docker Compose** type.
3. Enable **Isolated Deployments**. Keep automatic deployment off for the initial rollout.
4. Add **ERPNEXT_VERSION=v16.50.0** and a long, unique **DB_PASSWORD** in Dokploy's Environment tab. This password is never stored in Git.
5. Verify the DNS A record for **erpnext.benuel.dedyn.io** points to the intended Hetzner server; then add this hostname through Dokploy Domains, pointing to **frontend:8080** with HTTPS.
6. Review **Preview Compose** and only then deploy.
7. After healthy MariaDB and completed configurator, create the ERPNext site **erpnext.benuel.dedyn.io** once using the backend terminal.
8. Verify sign-in, authorization, persistent volumes, backup/recovery and operational monitoring.

**Never run docker compose down -v, delete active database volumes, or assume changing DB_PASSWORD in Dokploy rotates the initialized MariaDB password.**

## Official references

- https://github.com/frappe/frappe_docker
- https://github.com/frappe/frappe_docker/blob/main/docs/04-operations/01-site-operations.md
- https://docs.dokploy.com/docs/core/docker-compose
- https://docs.dokploy.com/docs/core/docker-compose/domains
