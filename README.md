# Infisical Coolify

Git-backed Coolify Compose deployment for self-hosted Infisical.

This repo is intended for a side-by-side staging migration from the old Coolify template that used `infisical/infisical:latest-postgres`. Do not replace production in place until the staging service has been restored and tested.

## What This Compose Does

- Pins Infisical with `INFISICAL_IMAGE` instead of using a stale floating `latest-postgres` image.
- Uses an explicit public `SITE_URL`.
- Keeps Postgres and Redis as sibling services.
- Generates new `ENCRYPTION_KEY` and `AUTH_SECRET` defaults for fresh installs, while allowing explicit migration overrides.
- Stores state in named Docker volumes: `pg-data` and `redis-data`.

## Coolify Environment

For a brand-new instance, Coolify can generate the root secrets and database credentials from the compose magic variables. You can deploy with only the image pin and database name customized:

```dotenv
INFISICAL_IMAGE=infisical/infisical:v0.160.5
SITE_URL=https://infisical.example.com
SERVICE_USER_POSTGRES=<Coolify generated or chosen username>
SERVICE_PASSWORD_POSTGRES=<Coolify generated or chosen password>
POSTGRES_DB=infisical
```

For migration, set `ENCRYPTION_KEY` and `AUTH_SECRET` explicitly from the old service before restoring the database. They override the generated defaults. Generating new values can make restored secrets unreadable.

Coolify must be `v4.0.0-beta.411` or newer for Git-based Compose magic environment variables.

## Staging Migration Plan

1. Create a new Coolify Compose resource from this repo.
2. Assign a staging domain, for example `https://infisical-staging.example.com:8080`.
3. Set `SITE_URL` to the public URL without the routing port, for example `https://infisical-staging.example.com`.
4. Set `ENCRYPTION_KEY` and `AUTH_SECRET` from the existing production service.
5. Backup the current production database with `pg_dump`.
6. Restore that dump into the new staging Postgres service.
7. Deploy and watch the backend logs for successful migrations.
8. Verify:
   - `/api/status` returns OK.
   - UI login works.
   - Existing projects and machine identities are visible.
   - Universal Auth login works.
   - `GET /api/v4/secrets` works with a valid machine identity token.
9. Cut over the production domain only after staging passes.

Keep the old Coolify service stopped but not deleted until production has been verified.

## Notes

- Keep `.env` files and database dumps out of git.
- Prefer pinned image tags over `latest`.
- Redis can be rebuilt if needed; Postgres plus `ENCRYPTION_KEY` are the critical pieces.
