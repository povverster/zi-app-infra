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

## Database migrations

Start PostgreSQL here, then apply migrations from the sibling API repository using
its [migration instructions](../zi-app-api/README.md#apply-pending-migrations).
API startup does not apply them automatically.

The manual-trade stage requires `AddManualTradeEntry` after the existing identity
migration. It preserves existing data; do not delete tables or the Docker volume.
See the [trade guide](../zi-app-api/docs/trading/manual-trade-entry.md#exchange-rate-handoff-and-migrations)
for the new pending-rate/correction schema and downgrade restrictions. Successful
API integration tests use separate disposable databases, not your local database.

## Ownership

This repository will eventually own production composition, reverse proxy, TLS,
backups, monitoring, and deployment configuration. API and web Dockerfiles remain
in their respective source repositories.
