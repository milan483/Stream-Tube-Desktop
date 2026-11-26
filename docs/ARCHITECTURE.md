# Architecture

Stream Tube is a native Rust desktop client. The application uses GPUI via `gpui-kit` for its interface, libmpv for playback, SQLite for local state, and direct network requests for YouTube and optional third-party services. It does not embed a browser for the main interface.

See the [README](../README.md) for user-facing behavior and [Cargo.toml](../Cargo.toml) for dependencies. This map describes the architecture introduced by the upstream 0.7.4-alpha revision; verify details against the current source before changing it.

## Component map

| Component | Responsibility |
| --- | --- |
| [`src/main.rs`](../src/main.rs) | Application startup, display/window initialization, storage, and single-instance handling |
| [`src/ui/`](../src/ui/) | Application state, navigation, pages, reusable widgets, cards, thumbnail caching and picture-in-picture |
| [`src/api/`](../src/api/) | YouTube Innertube and web requests, response parsing, stream extraction, authentication and third-party services |
| [`src/player/`](../src/player/) | Playback state and controller, libmpv runtime bindings, rendering, previews, Shorts preloading and streaming proxy |
| [`src/store.rs`](../src/store.rs) | SQLite database, settings, subscriptions, profiles, playlists, history and cached state |
| [`src/downloads.rs`](../src/downloads.rs) | Media downloads and optional external tools |
| [`src/feed.rs`](../src/feed.rs), [`src/recommend.rs`](../src/recommend.rs) | Local feed generation and recommendations |
| [`src/import.rs`](../src/import.rs) | Import and export of supported subscription and backup formats |
| [`src/i18n.rs`](../src/i18n.rs), [`src/i18n/locales/`](../src/i18n/locales/) | Interface translation and embedded language catalogs |
| [`src/locale.rs`](../src/locale.rs) | YouTube content language and region |
| [`src/display.rs`](../src/display.rs), [`src/links.rs`](../src/links.rs) | Display integration and deep links |
| [`src/updater.rs`](../src/updater.rs) | Upstream release discovery and application updates |

## Data and control flow

- Browse and search: GPUI page state requests data through the API layer, which parses responses into application models. Local preferences, history and caches are managed by the store.
- Playback: stream extraction resolves media and caption sources; the playback controller manages libmpv and exposes state to the UI. The local proxy supplies appropriate byte ranges or rewritten HLS playlists when needed.
- Account access: TV-device authorization and optional browser-based YouTube sessions serve different account operations. Browser sign-in uses a dedicated profile or an explicit cookie import. Credentials are sensitive even if stored locally.
- Updates: release discovery queries upstream; downloaded Windows application updates must pass the available integrity checks before execution. Linux application updates open the release page rather than using the Windows installer path.

## Boundaries and portability

The streaming proxy binds to loopback and requires a per-instance access token. The database and account session are local trust boundaries. Do not expose secrets in logs, tests, diagnostics or exports. Third-party APIs are separate from YouTube account authentication. External executables, native libraries and downloaded artifacts require provenance and integrity consideration.

GPUI and native libmpv interfaces introduce platform dependencies. Windows uses platform-specific libraries and an explicitly patched vendored GPUI Windows backend for bidirectional text. See [the vendor patch notes](../vendor/gpui-pre-windows/PATCHES.md). Linux packages and the Windows installer live under [packaging](../packaging/) and [installer](../installer/); the [Nix flake](../flake.nix) also defines a Linux package and development shell. macOS support is unofficial.

YouTube's undocumented interfaces, anti-bot behavior and changes to stream formats are external failure sources. Static inspection cannot establish playback reliability, GPU compatibility or account safety. Use targeted tests and native acceptance checks for affected behavior.
