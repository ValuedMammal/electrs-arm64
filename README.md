# electrs-arm64

Builds the [Blockstream/electrs](https://github.com/Blockstream/electrs) binary
(esplora fork) natively for linux/aarch64 using a GitHub-hosted `ubuntu-24.04-arm`
runner, then stages the binary as a release asset in this repo.

## Purpose

[electrsd](https://github.com/rust-bitcoin/corepc/tree/master/electrsd)'s `build.rs`
downloads an x86-64 electrs binary at build time, which doesn't work on arm64 runners.
CI workflows that run on arm64 can instead download the pre-built binary from this
repo's releases and point `ELECTRS_EXE` at it.

## Workflow

[`.github/workflows/electrs-esplora.yml`](.github/workflows/electrs-esplora.yml)
is triggered manually via `workflow_dispatch`. It:

1. Checks out `Blockstream/electrs` at a pinned commit
2. Builds `electrs` with `cargo build --locked --release` on a native arm64 runner
3. Publishes the binary as a release asset under the `electrs-bins` tag

To update the pinned commit, change `ELECTRS_COMMIT` in the workflow file and
re-run the workflow.

## Using the release asset in CI

The release asset is a zip archive. Download and extract it before setting
`ELECTRS_EXE`:

```yaml
- name: Download electrs (arm64)
  env:
    ELECTRS_COMMIT: 027e38d3ebc2f85b28ae76f8f3448438ee4fc7b1
  run: |
    curl -L -o electrs.zip \
      https://github.com/ValuedMammal/electrs-arm64/releases/download/electrs-bins/electrs_aarch64_linux_esplora_${ELECTRS_COMMIT}.zip
    unzip electrs.zip
    echo "ELECTRS_EXE=$PWD/electrs" >> "$GITHUB_ENV"
    echo "ELECTRSD_SKIP_DOWNLOAD=1" >> "$GITHUB_ENV"
```

`ELECTRSD_SKIP_DOWNLOAD=1` tells electrsd's `build.rs` not to attempt its own
download, and `ELECTRS_EXE` points it at the extracted binary.
