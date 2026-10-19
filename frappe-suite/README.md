# CETECH Frappeverse — v16 multi-app trial

Independent Frappe **v16** stack for CETECH, hosted through **new** Dokploy Compose service `Frappe` in project `Cetech-Erp-Frappeverse Apps`. This configuration is not a replacement for the existing deployed ERPNext Compose project.

## Included app sources

The `apps.json` manifest contains ERPNext, HRMS, Learning (LMS), Insights, CRM, Helpdesk, Lending, Raven, and their Payments/Telephony dependencies. These are *branch candidates*, not yet a tested release lock. Their branches should be resolved to immutable commit IDs and tested together before production.

Frappe itself is supplied by the official `frappe/build:version-16` layer.

## Build first: GitHub Actions

1. In the GitHub repository, open **Actions > Build Frappeverse v16 image > Run workflow**.
2. Workflow clones `frappe/frappe_docker`, builds the official `images/layered/Containerfile` with our `apps.json` passed as a BuildKit secret, and pushes an image to GHCR.
3. The image tag for the run is `ghcr.io/ben-001-sys/cetech-frappeverse:run-<github-run-id>`.
4. The build may fail if app dependencies clash; **do not deploy a partial or failing image**. Fix dependencies then rebuild.
5. Make the GHCR package pullable by Dokploy using a public package setting or registry credentials. Never disclose registry tokens in chat.

## Configure Dokploy

- Project: `Cetech-Erp-Frappeverse Apps`
- Service: `Frappe` (Docker Compose, **not** Docker Stack)
- Provider: GitHub, repository: `Ben-001-sys/cetech-erp`, branch `main`
- Compose Path: `./frappe-suite/docker-compose.yml`
- Enable **Isolated Deployments**.
- Disable automatic deployments and review **Preview Compose**.
- Add **FRAPPE_SUITE_IMAGE** (successful run tag), a fresh **SUITE_DB_PASSWORD**, and optional performance settings from `.env.example`.
- In **Domains**, point a **new hostname** to service `frontend`, port `8080`, and enable HTTPS. Do not reuse `erpnext.benuel.dedyn.io`.
- Do not add host `ports:` or deploy a second Traefik.

## First deployment and site setup

Only after image build succeeds, resource capacity is verified, registry pulling works, and a new hostname has been chosen:

1. Deploy Compose in Dokploy and inspect all containers, especially `configurator`, `backend`, `redis`, and workers.
2. Verify the backend image contains `frappe` and every app under `/home/frappe/frappe-bench/apps`.
3. Create the **new** Bench site *once* with the new hostname, against only the **new** MariaDB database. Use a strong independent Administrator password.
4. Install applications **with verified dependency order**, using `bench --site <new-hostname> install-app <app>` from the new backend, checking each install and frontend page. Do not assume bundling the code means the site has all apps installed.
5. Enable and test the scheduler, regular users, permissions, assets, websockets, site backups and full disaster-recovery restore.
6. Before future image upgrades: back up database/files/site configuration and encryption keys, roll out a pinned new image, run `bench --site <site> migrate`, and validate.

## Safety and compatibility warning

- Do not use this configuration to overwrite the existing root `docker-compose.yml`, deploy to the prior project, or share a Docker named volume.
- App `main` / `develop` branches can change unexpectedly. The current `payments` `develop` branch targets Frappe 17, so this manifest **explicitly uses `payments:version-16`**.
- The official images are based on Python 3.14/Node 24 for Frappe v16.
- No install success or performance claim is made until image build, site installs, and workflow tests pass.

## References

- https://frappe.github.io/frappe_docker/02-setup/02-build-setup.html
- https://frappe.github.io/frappe_docker/11-how-to/06-automated-builds-and-deployment.html
- https://docs.dokploy.com/docs/core/docker-compose/domains
