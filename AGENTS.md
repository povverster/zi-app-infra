# ZiApp Infrastructure: agent guide

## Purpose and repository boundaries

This repository owns local environments and eventual deployment operations for
ZiApp. Application source and Dockerfiles belong in the sibling
[API](../zi-app-api/AGENTS.md) and [web](../zi-app-web/AGENTS.md) repositories.
Keep the three repositories separate; deployment should compose their built images.

The product is a multi-user investment tracker with private portfolios, FIFO UAH
tax calculations, and English/Ukrainian/Russian UI. Its database and authentication
keys must survive routine service restarts and deployments.

## Working agreement

- Inspect Git status, current Compose files, README, and related API/web contracts
  before changes. Preserve unrelated work and make one verifiable stage at a time.
- Keep LF line endings and follow `.editorconfig` and `.gitattributes`.
- Create `.env` from `.env.example` only when absent; preserve existing local
  values. Keep real secrets, database dumps, TLS private keys, and authentication
  keys out of Git and logs. Example credentials are for local development only.
- Preserve the API's local HTTP port `5050` and HTTPS port `5051`.
  Local PostgreSQL defaults to port `5432`, configurable via `POSTGRES_PORT`.
- Keep EF Core schema migrations in the API repository. Infrastructure runs the
  documented migration procedure against an identified target database.
- The backend generates UUIDv7 entity keys stored as PostgreSQL `uuid`.
  A UUID generation change alone does not require rebuilding the database.
- Inspect the exact database/volume and current user authorization before any
  destructive reset. Routine stop/restart commands must preserve stored data.
- Keep API and browser requests on the same origin in production to support
  the existing HTTP-only cookie and CSRF authentication flow.
- Commit or push only when requested; verify and commit each repository separately.
- Update this guide and operational documentation after each completed stage.
  Distinguish configuration present in Git from services actually running or deployed.
  The roadmap does not authorize deploying every remaining stage automatically.

## Changelog maintenance

- Update [CHANGELOG.md](CHANGELOG.md) under `Unreleased` in the same task as each
  notable completed feature, behavior change, fix, or security improvement.
- Use the relevant Keep a Changelog categories: `Added`, `Changed`, `Deprecated`,
  `Removed`, `Fixed`, and `Security`. Omit empty categories and describe the
  effect for users or developers rather than copying commit messages.
- Document breaking changes and any required configuration or migration steps.
  Minor formatting edits do not need separate entries.
- Keep future work in this guide's development checklist, not in the changelog.
  For changes spanning repositories, update each affected repository's changelog.
- Move unreleased entries into a version/date section when an actual release is
  made. Do not invent historical releases or treat a commit as a release.
  A changelog update alone does not authorize tagging, publishing, or deployment.

## Development progress

Baseline inspected on 2026-09-05, at commit `09f7ca7`.
Annual-preparation/configured-tax handoff updated on 2026-10-04: infrastructure
documentation only; the API has additive migrations. No Compose/service/volume changes or
user-database migration were performed. Infrastructure milestones remain unchanged.

Frontend-foundation handoff updated 2026-10-05: README now describes the three-
terminal local workflow and web-only checks. The app shell is implemented in
`zi-app-web`; real browser authentication is next. No Compose, volume, service
configuration or database change was needed. Full local login acceptance remains open.

- [x] Separate infrastructure Git repository and LF conventions.
- [x] Local PostgreSQL Compose service: `postgres:18.6`, configurable local
  environment, persistent `postgres_data` volume, and readiness health check.
- [x] `.env.example`, ignored local `.env`, and startup/stop documentation.
- [x] CI validation of `compose.dev.yml`.
- [ ] Full local API/web composition. The current Compose file runs PostgreSQL only.
- [ ] Production deployment, reverse proxy/TLS, backups/restore, monitoring,
  and persistent ASP.NET Core Data Protection keys.

## Remaining development steps

Current cross-repository priority (2026-10-05): authentication UI in
`zi-app-web`, whose foundation is now complete. Follow its ordered product-screen
stages afterward. The API's user-configured
tax reporting is implemented; specialist review is not a blocker for that product
workflow. Infrastructure steps below remain unchanged and are not completed by
documentation updates. See the configured-tax migration handoff at the end.

1. [ ] Complete local acceptance across all three repos. The README now documents
   database startup, controlled migration/bootstrap references, API start and
   frontend start. The web uses localhost:5173 with a same-origin proxy to 5050.
   Real browser login/CSRF verification awaits the next authentication UI stage;
   isolated frontend proxy tests do not satisfy it.
   Add optional API/web Compose services when useful without breaking the existing
   database-only workflow.
2. [ ] Image build/publishing and production Compose: use API/web Dockerfiles from
   their source repos, versioned release images, runtime secrets, health checks,
   and persistent storage. Verify the composed app in a disposable environment.
3. [ ] Reverse proxy and TLS: same-origin web/API routing, certificate renewal,
   trusted forwarded headers, HTTPS cookies, and deliberate database network
   exposure. Verify authentication through the actual proxy.
4. [ ] Authentication key persistence: persist ASP.NET Core Data Protection keys,
   protect access, and share compatible keys/configuration across any API replicas.
   Verify login sessions survive intended restarts and replica routing.
5. [ ] Database operations: documented migration ordering, database backup schedule,
   retention, protected off-host storage, and a restore procedure. Prove restore
   works in a separate database and verify application data after restoration.
6. [ ] Deployment and recovery workflow: pre-deploy checks/backups, controlled
   migration execution, versioned rollout, health verification, and recovery
   instructions compatible with schema changes. Rehearse on staging.
7. [ ] Monitoring and alerts: service/database health, useful logs without secrets,
   disk/storage capacity, backup failures, and certificate expiry.
8. [ ] Production acceptance: verify user isolation, login/logout/CSRF, portfolio
   and reporting workflows once implemented, restore/recovery, and operational
   handoff documentation before releasing access to other users.

Steps 3-7 are release prerequisites, not evidence that production is already ready.

## Local commands and verification

Run Compose commands from this repository root. Create the local environment
file once; this guard avoids overwriting an existing configuration:

```powershell
if (-not (Test-Path .env)) { Copy-Item .env.example .env }
docker compose --env-file .env.example -f compose.dev.yml config --quiet
docker compose -f compose.dev.yml up -d postgres
docker compose -f compose.dev.yml ps
```

Stop without deleting the database volume:

```powershell
docker compose -f compose.dev.yml down
```

Validate changed Compose configuration with the example environment as CI does.
For runtime configuration changes, also start the affected services and inspect
health/readiness. Review logs locally without exposing credentials. Database
persistence changes need a restart/restore check using appropriate test data.
For documentation-only changes, verify paths, commands, LF endings, and
`git diff --check`; no service restart is required.

Apply migrations from the API repository root, after confirming the target.
For non-default database settings, set `ConnectionStrings__Database` consistently
for the API and its design-time migration tooling:

```powershell
dotnet tool restore
dotnet ef migrations list --project src/ZiApp.Infrastructure --startup-project src/ZiApp.Api
dotnet ef database update --project src/ZiApp.Infrastructure --startup-project src/ZiApp.Api
```

The API does not migrate the development database on startup. Testcontainers used
by backend integration tests create separate disposable databases; passing those
tests does not mean the user's local database has been migrated.

As of 2026-09-20, manual trade entry requires the API's `AddManualTradeEntry`
migration after `AddIdentityAuthentication`; NBU resolution additionally requires
`20260920191527_AddNbuExchangeRates`. See the
[trade migration contract](../zi-app-api/docs/trading/manual-trade-entry.md#exchange-rate-handoff-and-migrations).
It preserves existing records and adds pending FX/correction audit support.
Do not reset the volume. Downgrade refuses to erase correction history or pending
rates; the NBU migration's downgrade also refuses to erase response provenance or
rate-resolution audit history. Prefer a forward migration.

See the [NBU operations/acceptance guide](../zi-app-api/docs/exchange-rates/nbu-exchange-rates.md).
On-demand rate fetching needs outbound HTTPS to `bank.gov.ua` without an API key.
Retries/timeouts are bounded; there is no scheduled import/backfill service to deploy.
Backend tests use fixed responses plus disposable PostgreSQL, not live NBU requests.
No Compose or volume changes are needed. This documentation update does not mean
the user's database has been migrated or the infrastructure roadmap has advanced.

As of 2026-10-02, splits/holdings and current trade readers also require
`20261002125349_AddSplitManagement` after `AddNbuExchangeRates`. See the
[split migration/acceptance guide](../zi-app-api/docs/holdings/splits-and-holdings.md).
The migration preserves existing split IDs/ratios, backfills ordering keys, and
does not invent old provenance. Downgrade blocks loss of new audit history.
Drain older API writers before upgrading: all quantity-changing writers must
use the new shared/exclusive transaction advisory lock before portfolio locks.
Apply the forward migration, then run only updated writers. No new services,
network dependencies, credentials or volume changes are needed for this stage.
Holdings reads do not contact NBU. Tests use disposable databases, not the user's DB.

As of 2026-10-03, saved draft reports require
`20261002140835_AddDraftTaxReports` after `AddSplitManagement`. Read the
[report migration/acceptance guide](../zi-app-api/docs/reports/draft-tax-reports.md).
It adds nullable immutable-snapshot metadata and widens calculated match columns
to unconstrained numeric; source-trade/rate precision and existing IDs/data remain
unchanged. Legacy runs are preserved without invented snapshot backfills.
Downgrade refuses to erase saved drafts or narrow calculated values with data loss.

Use a controlled schema window: stop/drain old API writers, back up and confirm
the target database, apply forward migrations, then start updated instances.
Reports save a single Repeatable Read view of inputs and never call NBU.
No new services, secrets, Compose/volume changes or background jobs are needed.
Stored snapshots include private financial history: backups must retain and protect
these alongside source/audit rows. Passing disposable-DB tests does not apply this
migration to the user's development database or complete an infrastructure stage.

## Filing-research handoff

The [2025 review](../zi-app-api/docs/reports/ua-2025-filing-readiness.md) records
scope decisions, official-source findings and open legal/form/rounding gates.
The user approved a separate annual preparation summary while retaining current
one-portfolio drafts. The [annual preparation API](../zi-app-api/docs/reports/annual-preparation-drafts.md)
is now implemented and requires `20261003152645_AddAnnualPreparationDrafts`
after `AddDraftTaxReports`. It adds a separate immutable annual snapshot table,
owner FK/index/checks and no changes to existing report/source data. Downgrade
refuses to erase saved annual drafts. Apply forward to a confirmed, backed-up target
in a controlled schema window; do not reset the database or volume.

Annual summaries include outside-app financial inputs and prior-loss
evidence references. Treat them as private financial data in backup/logging design,
with owner isolation and preserved snapshots. Research has not enabled official
filings, tax payable, automated submission or a scheduler. No new services,
credentials or network dependencies are needed. Only disposable test databases
were migrated during API verification; the user's local database remains unchanged.

The added personal `zi-samples` workbooks were inspected read-only by the API
task; see the [sample audit](../zi-app-api/docs/domain/spreadsheet-sample-audit.md).
Do not copy these private source files into deployment images, Git or backups
configured implicitly for this stage. Their availability is not a service dependency.

## Configured-tax handoff, 2026-10-04

The [configured-report contract](../zi-app-api/docs/reports/configured-tax-reports.md)
adds year-specific private settings revisions and immutable configured reports.
It preserves signed negative losses with zero taxes on loss years. Dividends are
deferred; their rate alone is stored. Official statutory/form validation remains
separate and does not block this user-configured workflow.

Apply `20261004144649_AddConfiguredTaxReports` after
`20261003152645_AddAnnualPreparationDrafts` to a confirmed, backed-up target in
the established controlled schema window. It adds `tax_settings_revisions` and
`configured_tax_reports` with restrictive source/settings references. Existing
ledger/report data is unchanged. Downgrade refuses to erase either table's data.
No reset, Compose change, network dependency, secret or background service is needed.
Include settings revisions and private report snapshots in backup/restore design.
The API suite used only disposable test databases; the user's DB was not migrated.

For another AI: read all three AGENTS guides and the linked contract; do not
repeat the superseded specialist-review/review-package step. Follow the shared
authentication-UI priority, preserve volumes and apply no user DB migration
without confirming the target. Commit/push only when requested.

## Frontend foundation handoff, 2026-10-05

Read the [web guide](../zi-app-web/AGENTS.md) and
[foundation decisions](../zi-app-web/docs/frontend-foundation.md). Node 22.23.2,
npm 10.9.8, `npm ci` and `npm run dev` start its local app on port 5173.
`API_PROXY_TARGET` is a web-server-only origin setting, defaulting to
http://localhost:5050. Keep secrets out of browser VITE_* variables.

The frontend test suite starts isolated loopback servers on 4173/5510; it neither
uses nor migrates PostgreSQL. Real backend connection checks are read-only and
manual. A healthy DB check does not prove current migrations or account access.
No frontend production Dockerfile, TLS proxy, container composition, secret/key
persistence, release or deployment has been added.
