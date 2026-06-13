# electrs-arm64

Builds the [Blockstream/electrs](https://github.com/Blockstream/electrs) binary
(esplora fork) natively for linux/aarch64 using a GitHub-hosted `ubuntu-24.04-arm`
runner, then stages the binary as a release asset in this repo.

## Purpose

[electrsd](https://github.com/RCasatta/electrsd)'s `build.rs` downloads an
x86-64 electrs binary at build time, which doesn't work on arm64 runners. CI
workflows that run on arm64 can instead download the pre-built binary from this
repo's releases and point `ELECTRS_EXE` at it.

## Workflow

[`.github/workflows/electrs-esplora.yml`](.github/workflows/electrs-esplora.yml)
is triggered manually via `workflow_dispatch`. It:

1. Checks out `Blockstream/electrs` at a pinned commit
2. Builds `electrs` with `cargo build --locked --release` on a native arm64 runner
3. Publishes the binary as a release asset under the `electrs-bins` tag

To update the pinned commit, change `ELECTRS_COMMIT` in the workflow file and
re-run the workflow.
