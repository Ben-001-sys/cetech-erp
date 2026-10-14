# CETECH ERPNext — Dokploy deployment runbook

**Status: deployed and site installed 2026-10-14; post-install verification pending.**

**2026-10-14 installation evidence:** Existing Dokploy Compose project `cetecherpnext-erpfrontend-otl3fx`, with healthy MariaDB and successful configurator. Bench `new-site erpnext.benuel.dedyn.io --db-root-username root --mariadb-user-host-login-scope='%' --install-app erpnext` completed Frappe and ERPNext DocTypes at 100%. Bench reported `SystemSettings.enable_scheduler is UNSET` and `Scheduler is disabled`. Do not run `new-site` again. Next: validate list-apps, enable scheduler, check HTTPS/login, and configure backups. Target: existing Hetzner server running Dokploy. Intended hostname: **erpnext.benuel.dedyn.io** (confirmed by deployment owner; public DNS and certificate are not yet verified).

## 1. Inspect the existing server (read-only)

Connect to the Hetzner host (do not install a second Dokploy or Nginx):

```bash
hostnamectl
nproc
free -h
df -hT
docker version
docker compose version
docker ps --format 'table {{.Names}}\t{{.Status}}\t{{.Ports}}'
docker system df
```

Confirm available RAM/CPU/disk, existing deployed applications, Docker networks, and where persistent volumes will reside before starting a resource-intensive ERP stack.

## 2. Connect source in Dokploy

1. Inside the CETECH ERPNext project / production environment, create one **Compose** service using **Docker Compose**, **not Docker Stack**.
2. Source: **GitHub**; repository: **Ben-001-sys/cetech-erp**; branch: **main**; Compose path: **./docker-compose.yml**.
3. Deploy target: the existing Hetzner Dokploy server. Enable **Isolated Deployments** to attach the required private network and Dokploy Traefik correctly.
4. Keep **Auto Deploy disabled** during the first rollout.
5. In the service's Environment tab, add **ERPNEXT_VERSION=v16.50.0** (revalidate release before deploying) and **DB_PASSWORD** with a strong unique generated value. The actual password must not be committed to GitHub. Optional variables are documented in .env.example.
6. Verify the DNS A record for **erpnext.benuel.dedyn.io** points to the intended Hetzner public IPv4 address (and verify any AAAA record targets the same server). In the **Domains** tab add **erpnext.benuel.dedyn.io**, selecting **frontend** and **internal/container port 8080**, with HTTPS / Let's Encrypt via Dokploy. Do **not** put `https://` in the domain field.
7. Use **Preview Compose** and verify secrets are resolved, named volumes remain, and labels/networks point to frontend. Avoid sharing screenshots containing revealed secrets.

The Compose file intentionally has no host-published ports and no extra Traefik. Hostname routing via `FRAPPE_SITE_NAME_HEADER: $host` works when the Bench site name matches the incoming hostname. **Do not set FRAPPE_SITE_NAME_HEADER unless the site name differs from the host**.

## 3. Validate and deploy

On a trusted machine with Docker Compose installed, for local syntax/config validation only:

```bash
cp .env.example .env
# Replace the example DB_PASSWORD in this local untracked file with a test-only value.
docker compose --env-file .env -f docker-compose.yml config --quiet
```

Save the real variables in Dokploy and click **Deploy**. Watch logs/status: db must become healthy; redis-cache and redis-queue start; configurator exits **0**; backend, frontend, websocket, scheduler, queue-short, and queue-long should run. A completed configurator is normal; it is a one-time setup service.

## 4. Initialize exactly once

After database health and successful configurator completion, open the **backend** container terminal in Dokploy. Inspect before creating anything:

```bash
ls -la /home/frappe/frappe-bench/sites
cat /home/frappe/frappe-bench/sites/common_site_config.json
bench list-sites
```

Only when no prior site exists, and the hostname has been approved, run in the backend container:

```bash
bench new-site erpnext.benuel.dedyn.io \
  --mariadb-user-host-login-scope='%' \
  --install-app erpnext
```

Enter the MariaDB root password and the desired ERPNext Administrator password at the secure interactive prompts. Do not pass passwords as shell arguments, commit them, or publish them in logs. If Dokploy's terminal does not support interactive prompts, use a secure interactive `docker compose exec -it backend ...` session on the server.

## 5. Acceptance tests

1. Confirm HTTPS and ERPNext Administrator login.
2. Check basic company setup, create and remove test records; verify authorization using both positive and negative role tests.
3. Check scheduler, WebSocket/realtime and background job logs.
4. Restart an application container; verify data, files and configuration persist.
5. Configure verified logical site backups, encrypted off-server storage and a test restore **before** live business data.
6. Set monitoring/alerts and an upgrade/recovery procedure.

## Troubleshooting

- 404 before site creation: expected; later verify domain, host header and site name.
- configurator exited 0: correct. A nonzero exit needs log inspection.
- Traefik 502: inspect frontend:8080, Dokploy isolated networking and internal services.
- MariaDB authentication error: confirm original database-root password and volume; changing DB_PASSWORD in Dokploy does **not** rotate a populated MariaDB database.
- Do not use `docker compose down -v`.

References:
- https://github.com/frappe/frappe_docker
- https://github.com/frappe/frappe_docker/blob/main/docs/04-operations/01-site-operations.md
- https://docs.dokploy.com/docs/core/docker-compose
- https://docs.dokploy.com/docs/core/docker-compose/domains

## 6. Post-install verification (2026-10-14)

Run in the currently active backend container `cetecherpnext-erpfrontend-otl3fx-backend-1`:

```bash
bench --site erpnext.benuel.dedyn.io list-apps
bench --site erpnext.benuel.dedyn.io enable-scheduler
bench --site erpnext.benuel.dedyn.io scheduler status
```

The scheduler was disabled immediately after installation; mark as enabled only after the status output confirms it. Verify HTTPS and Administrator sign-in separately. Preserve all named volumes and do not recreate the site.
