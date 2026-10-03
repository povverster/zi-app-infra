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

NBU rate resolution additionally requires `20260920191527_AddNbuExchangeRates`
after `AddManualTradeEntry`. This preserves existing rates/trades and adds
provenance and resolution audit fields. Downgrade refuses to erase that new history.
Use the same migration procedure, without resetting the database.

The API needs outbound HTTPS to `bank.gov.ua` for on-demand rate requests, with
no API key. There is no rate scheduler or automatic backfill to configure.
See the [NBU guide](../zi-app-api/docs/exchange-rates/nbu-exchange-rates.md)
for exact-date behavior, errors, and acceptance checks. Automated tests use fixed
NBU responses. Existing Compose services and volumes are unchanged.

## Split/holdings upgrade

The API additionally requires `20261002125349_AddSplitManagement` after
`AddNbuExchangeRates`. It preserves existing splits/trades/rates/reports and adds
split ordering/provenance/correction storage. Downgrade refuses to erase new
split history; use forward migrations without deleting the database.

Drain older API writers before upgrading: this stage coordinates trade and split
changes with a shared/exclusive ledger lock that older writers do not take.
Apply the migration to the confirmed target, then run only updated writers.
Use the [split/holdings acceptance guide](../zi-app-api/docs/holdings/splits-and-holdings.md).
No Compose/volume changes or extra network configuration are needed; holdings
GET does not fetch rates. Passing integration tests does not migrate your local DB.

## Saved draft report upgrade

The later [2025 filing-readiness research](../zi-app-api/docs/reports/ua-2025-filing-readiness.md)
remains a legal-research baseline, not filing approval. Its separate annual
preparation API is now implemented; see the additional upgrade below. The
existing saved-report upgrade instructions remain unchanged.

The API requires `20261002140835_AddDraftTaxReports` after `AddSplitManagement`.
It preserves source data and existing runs, adds nullable report snapshot metadata
and widens calculated match columns to retain full .NET decimal precision.
Legacy runs are not backfilled with guessed historical inputs. Downgrade refuses
to erase saved snapshots or narrow calculated values with precision/range loss.

Stop/drain old API writers, back up the confirmed target database and apply the
forward migration before starting updated instances. Use the
[report acceptance guide](../zi-app-api/docs/reports/draft-tax-reports.md) to verify
saved history, exports, missing-rate handling and privacy. Reports are drafts,
not official filings, and never contact NBU during creation or reads.

No Compose/volume changes or extra services are needed. Backups must preserve
private report snapshots together with their source/audit data. Automated tests
use disposable databases and do not migrate the user's development database.

## Annual preparation upgrade

The [annual preparation contract](../zi-app-api/docs/reports/annual-preparation-drafts.md)
requires `20261003152645_AddAnnualPreparationDrafts` after `AddDraftTaxReports`.
It adds only a separate owner-scoped snapshot table with its index/constraints.
Previous portfolio reports, trades and values remain unchanged. Downgrade refuses
to erase any saved annual drafts; use forward migrations without resetting data.

Confirm/back up the target and follow the same controlled migration procedure.
API startup and tests do not migrate the user's database. No new Compose service,
secret, external network dependency or volume is needed. Include annual snapshots,
outside-app amounts and evidence references in private-data backup/logging design.
All annual drafts remain non-filing-ready, with no payable or accepted loss deduction.

## Ownership

This repository will eventually own production composition, reverse proxy, TLS,
backups, monitoring, and deployment configuration. API and web Dockerfiles remain
in their respective source repositories.
