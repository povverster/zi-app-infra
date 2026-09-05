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

- Local PostgreSQL image from `postgres:18.6-alpine` to `postgres:18.6`.
- Standardized text files on LF line endings through Git and editor settings.
