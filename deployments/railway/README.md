# Autono Plane deployment

Plane Community 1.5.0 runs on Railway with PostgreSQL, Redis, RabbitMQ and a private S3-compatible Railway bucket. One instance supports multiple workspaces and memberships; tenants do not receive separate databases.

Build from the repository root using `deployments/railway/Dockerfile`. It extends the official versioned image without changing application code. The Supervisor configuration preserves upstream services, sends their logs to stdout, and explicitly sets Celery concurrency to two.

## Configuration

Use the upstream AIO environment contract in `deployments/aio/community/README.md`. Set `APP_PROTOCOL=https`, `SITE_ADDRESS=:80`, `PORT=80`, and `DOMAIN_NAME` to the served hostname. Use generated Railway references for database, cache, broker and bucket credentials. Keep `SECRET_KEY` and `LIVE_SERVER_SECRET_KEY` stable across deployments. Secrets belong in Railway, not this repository.

Set `ENABLE_SIGNUP=0` for invited users only and `DISABLE_WORKSPACE_CREATION=0` to permit workspace creation. These initialize instance settings; manage them in God mode after first boot. Initial administrator setup uses Plane's own setup flow, not direct database writes. Complete it before switching the public hostname.

Health check: `/api/instances/`. The image runs upstream migrations during startup. Keep all stateful services private. Persist PostgreSQL and RabbitMQ data; enable database backups before storing production data. Take and verify a backup before every release upgrade.

## Cutover and rollback

Validate authentication, two separate workspaces, membership isolation and uploads on the temporary Railway hostname. Then set `DOMAIN_NAME=plan.autono.co`, move that custom domain from the old service to Plane, and update its DNS records. Retain the old application, database and bucket for rollback. This deployment does not migrate data from It's a Plan.

SMTP and AI integrations are not provisioned by this configuration.
