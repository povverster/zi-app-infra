# Changelog

Notable changes to ZiApp infrastructure are recorded here, using the
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/) format.

The initial entries summarize completed development work from Git history.
They remain unreleased until assigned to an actual versioned release.
Planned work and development instructions are in [AGENTS.md](AGENTS.md).

## [Unreleased]

### Added

- Local PostgreSQL development service in `compose.dev.yml`, with configurable
  database settings and host port, a readiness health check, and persistent
  storage in the `postgres_data` volume.
- Development environment template, an ignored local `.env` file, and
  instructions for starting PostgreSQL and stopping it without deleting stored data.
- CI validation of the development Compose configuration using `.env.example`.
- An `AGENTS.md` guide covering repository ownership, local operations, migration
  coordination, deployment milestones, and changelog maintenance rules.

### Changed

- Documented the API's `AddSplitManagement` migration, preserved legacy split
  records, protected audit history and requirement to drain old API writers before
  adopting the new trade/split locking protocol. Compose and volumes are unchanged.

- Documented the API's `AddNbuExchangeRates` migration after manual trade entry,
  preserved existing data and protected audit history, outbound NBU HTTPS needs,
  and fixed-response testing. No new services, credentials, or volume reset.

- Documented the API's manual-trade migration requirement and data-preserving
  upgrade/rollback restrictions. Compose settings and existing volumes are unchanged.
- Local PostgreSQL image from `postgres:18.6-alpine` to `postgres:18.6`.
- Standardized text files on LF line endings through Git and editor settings.
