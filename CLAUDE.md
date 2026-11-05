# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A GitHub composite Action (`action.yml`) that installs the [`crustyimg`](https://github.com/jysf/crustyimg) CLI on a runner and adds it to `PATH`. There is no application code, build step, or lint config: the whole product is `action.yml`, plus `.github/workflows/self-test.yml` as its test suite.

## Testing

There are no local test commands. The action is validated only by the `self-test` workflow, which runs on push, PR, and `workflow_dispatch`. It uses `uses: ./` across a matrix of `ubuntu/macos/windows-latest` × `[v0.8.0, latest]`, and checks `--version`, that a pinned install reports exactly that version, `lint test-assets/clean.png`, and `optimize` producing an output file. To check a change, push it and read the workflow run (`gh run list` / `gh run watch`).

## Architecture (`action.yml`)

Steps run in this order, and later steps depend on earlier outputs:

1. **Resolve tag** (bash): normalizes `inputs.version` to a concrete `vX.Y.Z` tag. `latest` is resolved by following the `github.com/jysf/crustyimg/releases/latest` redirect with `curl -w '%{url_effective}'`. This is deliberate: it needs no API token, has no rate limit, and avoids a `curl | grep -m1` pipe that failed under `pipefail`. Bare `0.4.0` gets a `v` prefix. Anything that doesn't resolve to `v<digit>…` fails with `::error::`.
2. **Cache** (`actions/cache@v6`): the path is `runner.tool_cache/crustyimg/<tag>` and the key is `os-arch-tag`. Using the resolved tag (not `latest`) keeps the key stable.
3. **Install**: runs only on a cache miss. There are separate Unix (bash) and Windows (pwsh) steps, each running crustyimg's cargo-dist installer for that tag. `CRUSTYIMG_INSTALL_DIR` points at the cache path, and `CRUSTYIMG_NO_MODIFY_PATH=1` stops the installer from editing shell profiles. Checksum verification is done by the installer; the action does not download tarballs or hardcode asset names, so keep it that way.
4. **PATH**: appends `<install dir>/bin` to `$GITHUB_PATH` on both the cache-hit and install paths.

## Conventions

- Any cross-OS change must work on bash (Unix and Windows runners) and pwsh. Prefer `runner.tool_cache` expressions over shell path handling.
- Runs use `set -euo pipefail`. Avoid pipes whose early exit can break an upstream command.
- The `description` in `action.yml` must stay within the Marketplace's 125-character limit.
- `v1` is a moving major tag that points at the newest `v1.x` release. Exact tags such as `v1.0.4` are immutable pins. Releasing means cutting a `v1.x.y` tag and moving `v1`.
- If you change the `actions/cache` version, update the matching note in the README (it mentions Node 24 and the self-hosted runner minimum).
