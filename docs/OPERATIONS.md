# CETECH ERPNext — Operations and recovery

**Applies after initialization, not before.**

## Persistent data

Our Dokploy Compose configuration defines Docker named volumes:

- **sites:** Site configuration, encryption keys, public and private files, and local logical backups.
- **db-data:** MariaDB live business data.
- **redis-queue-data:** Redis queue persistence directory (not the authoritative transaction database).

Docker Compose normally prefixes them with the deployed application/project name. Record their exact runtime names with `docker volume ls` after deployment. Never use `docker compose down -v` or delete/replace persistent volumes during redeployments. Dokploy Volume Backups support named volumes, but a running MariaDB raw volume snapshot alone is not a consistent database backup.

## Logical application backup

After ERPNext site creation, enter the backend container and run:

```bash
bench --site erpnext.benuel.dedyn.io backup --with-files --compress
```

Schedule this operationally. Transfer backup archives **off Hetzner** to access-restricted, encrypted object storage; set retention policies and failure alerting. Protect the matching site configuration and encryption key as part of a complete recovery set. Dokploy's system backup alone does not back up ERPNext business data.

## Restore tests

Periodically test recovery in a separate isolated environment using matching ERPNext/Frappe software, the database backup, files and encryption key. Follow official Bench restore procedures and verify users, files, permissions and sample business transactions. Record the test outcomes and recovery time.

## Safe upgrade

1. Record baseline release and successful backup/restore evidence.
2. Test an image update on an isolated clone before production.
3. Update `ERPNEXT_VERSION` in Dokploy to a verified compatible tag; do not track `latest`.
4. Redeploy while keeping named volumes.
5. Run `bench --site erpnext.benuel.dedyn.io migrate` from the backend container if required by the upstream upgrade notes.
6. Confirm site access, assets, workers, scheduler, WebSocket and representative workflows.
7. An image rollback after DB migrations may be unsafe. Use a compatible verified backup for recovery if necessary.

## Security

Store the database root password in Dokploy's protected Environment settings or a secrets provider, never in GitHub. Changing the Docker environment setting after MariaDB's first initialization will not change the existing root password; follow MariaDB's password rotation procedure. Restrict Dokploy users, enable MFA, keep database/Redis private, and monitor backups, disk capacity, and container health.

References:
- https://github.com/frappe/frappe_docker
- https://docs.dokploy.com/docs/core/volume-backups
