# Repository development guide

Stream Tube is an MIT-licensed native YouTube client written in Rust with GPUI and libmpv. This repository is a fork of [upstream](https://github.com/Avaneesh-Inamdar/Stream-Tube-Desktop).

## Start here

- Read [README.md](README.md) for features, installation and build prerequisites, and [CONTRIBUTING.md](CONTRIBUTING.md) for upstream contribution conventions.
- Read [architecture](docs/ARCHITECTURE.md) and [development and verification](docs/DEVELOPMENT.md) for implementation boundaries, commands and current CI coverage.
- Inspect the current fork and upstream revisions, relevant issues and PRs before starting a change. Reuse upstream fixes rather than reimplementing them.

## Working on the repository

- Keep one scoped change per branch and PR. Preserve upstream history, unrelated work and the lockfile unless dependencies change.
- Reuse the existing UI helpers in `src/ui/widgets.rs` and `src/ui/mod.rs`; avoid unnecessary dependencies and broad formatting-only changes.
- Add regression tests for testable behavioral changes. For playback and UI changes, also verify the native application on the relevant platform. Record the tested revision, commands, outcomes and skipped checks in the task PR.
- Keep interface translation (`src/i18n.rs` and `src/i18n/locales/`) separate from YouTube content language and region (`src/locale.rs`).
- Keep platform-specific changes gated appropriately. Preserve compatibility with Windows and Linux; do not assume the unofficial macOS support is fully verified.

## Security and data

- Do not introduce telemetry or send account credentials, cookies or watch-history reports to unrelated services.
- Use a separate data directory for development and manual tests. Do not inspect or reuse a personal browser profile or real account credentials for automated tests.
- Preserve loopback-only streaming and the proxy's per-instance access token. Do not remove update or download integrity checks.
- Treat changes to authentication, network access, local storage, native bindings, downloads and release workflows as security-sensitive.

## Task continuity

Use the relevant GitHub issue and draft PR to record scope, decisions, revisions, verification, blockers and next steps. Reconcile this record with current Git and CI before resuming. Follow applicable authorization requirements; these instructions do not authorize external actions, publication, releases, or merging.
