# Development and verification

This guide supplements the [README build instructions](../README.md#building) and [contribution guide](../CONTRIBUTING.md). Run commands from the repository root in a dedicated task checkout. Do not install packages or modify a live installation merely to run a check.

## Build and test

Use a Rust toolchain compatible with the current lockfile. The README lists Rust 1.85 or newer for the 2024 edition; that is not a verified minimum for all locked dependencies. CI uses stable Rust. See the [Linux workflow](../.github/workflows/linux.yml) for build libraries and the README for runtime libmpv requirements. yt-dlp and FFmpeg are optional tools for additional playback and download capabilities.

```sh
cargo build --release --locked
cargo test --release --locked
```

The commands above match the Windows CI build and test steps. Linux CI currently builds and packages the application, but does not run the Rust test suite. Run the test command separately on Linux when the environment is available. The Nix package currently has `doCheck = false`; building the flake package is not equivalent to running tests.

Use targeted tests for the affected module and a full suite when practical. Existing inline Rust tests cover areas such as localization, stream parsing, updates and loopback proxy access. Do not automatically enable ignored tests that depend on a live browser, account or network.

The following are useful diagnostics when reviewing a change, but are not currently enforced by the inspected CI workflows:

```sh
cargo fmt --check
cargo clippy --locked --all-targets
```

Existing formatting or lint debt should be distinguished from regressions rather than fixed wholesale in an unrelated PR.

## Native manual testing

Use an isolated application data directory instead of the usual database. Example for a Linux shell:

```sh
test_data="$(mktemp -d)"
STREAMTUBE_DATA_DIR="$test_data" ./target/release/streamtube
```

This launches the application and may make network requests. Leave the temporary directory available for inspection until the test is complete; never substitute a user's actual profile or database for a disposable test profile. Do not sign in or import real cookies in routine smoke tests. Use separate, explicitly authorized procedures when account integration is the subject of testing.

Check relevant behavior: startup, window and theme handling, search, playback, seeking, captions, hardware decoding, fullscreen, audio, downloads, settings persistence and exit. Record platform, GPU/session type where relevant, tested revision, expected and observed results. A successful compile is not a native acceptance test.

## CI and releases

| Workflow | Purpose | Coverage |
| --- | --- | --- |
| [Windows build](../.github/workflows/windows.yml) | Windows compilation and Rust tests | Release builds and locked tests; installer/portable package on configured packaging runs |
| [Linux build](../.github/workflows/linux.yml) | Linux compilation and packaging | Ubuntu 22.04 runner; tarball, DEB, RPM, AppImage and DEB installation check |
| [Release](../.github/workflows/release.yml) | Version-tag validation and artifact production | Reusable Linux/Windows builds, SHA-256 sums, provenance attestations and draft release creation |

Do not push version tags, dispatch release workflows or publish releases for routine development. Upstream application update endpoints are still used by this fork; customized distribution requires an explicit decision about update channels.

The current CI baseline has no enforced Linux Rust test step, formatting or Clippy gate, or dependency security audit. Some Linux packaging inputs are mutable and desktop metadata validation is advisory. Review these separately without weakening existing checks. Native Wayland, GPU, playback and account behavior require additional acceptance evidence; macOS is not covered by these CI workflows.

## Verification records

Use the task issue or PR to record the base and head SHA, commands actually run, CI run links, successful and failed checks, and outstanding native tests. Distinguish upstream evidence on an identical commit from runs in this fork. Never describe a pending or skipped check as a pass.
