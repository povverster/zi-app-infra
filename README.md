# ZiApp infrastructure

Local development and deployment configuration for Zorjd Investments.

The application source stays in sibling repositories:

```text
zorjd-investments/
  zi-app-api/
  zi-app-web/
  zi-app-infra/
```

## Start local PostgreSQL

Create your untracked local environment file once:

```powershell
Copy-Item .env.example .env
```

Start the database:

```powershell
docker compose -f compose.dev.yml up -d postgres
docker compose -f compose.dev.yml ps
```

Stop services without deleting data:

```powershell
docker compose -f compose.dev.yml down
```

The default local connection string is:

```text
Host=localhost;Port=5432;Database=zi_app;Username=zi_app;Password=zi_app_local_dev
```

The defaults are development-only. Production secrets must come from the deployment
environment and must never be committed.

## Ownership

This repository will eventually own production composition, reverse proxy, TLS,
backups, monitoring, and deployment configuration. API and web Dockerfiles remain
in their respective source repositories.
