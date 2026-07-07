# setup-crustyimg

A GitHub Action that installs the [`crustyimg`](https://github.com/jysf/crustyimg) image CLI —
a fast, pure-Rust tool to **optimize, convert, and lint** images — and adds it to `PATH`.

It wraps crustyimg's official [cargo-dist](https://opensource.axo.dev/cargo-dist/) installer, so it
**detects the runner OS/arch and verifies the release sha256 checksum** for you, on
Linux, macOS, and Windows. It installs the *binary*, so every crustyimg command is then available in
your workflow — not just `lint`.

## Usage

```yaml
- uses: jysf/setup-crustyimg@v1
- run: crustyimg lint assets/          # or optimize / convert / …
```

Pin a version (recommended for reproducible CI):

```yaml
- uses: jysf/setup-crustyimg@v1
  with:
    version: v0.4.0        # a release tag, or "latest" (the default)
```

That's it — subsequent steps can call `crustyimg` directly.

### For image linting specifically

See [`jysf/crustyimg-action`](https://github.com/jysf/crustyimg-action), which builds on this
action to run `crustyimg lint` and turn findings into inline PR annotations.

## Inputs

| Input     | Default  | Description |
|-----------|----------|-------------|
| `version` | `latest` | crustyimg release to install: a tag like `v0.4.0` (or bare `0.4.0`), or `latest`. |

## How it works

1. Resolves `version` to a concrete release tag (so the cache key is stable).
2. Restores a cache of the install (keyed on OS/arch/tag), or, on a miss, downloads and runs the
   cargo-dist installer for that tag — which verifies the checksum before installing.
3. Adds the install directory to `PATH` for the rest of the job.

No inputs are required; `version: latest` works out of the box.

## Notes

- **Checksum verification** is handled by crustyimg's cargo-dist installer (`verify_checksum`); this
  action does not download raw tarballs or hardcode asset names, so it keeps working across releases.
- The install is cached under the runner tool-cache, so repeat runs skip the download.

## Maintainer / release notes

This repo ships the action + a 3-OS self-test workflow. Two outward steps remain for the maintainer,
and are intentionally **not** done automatically:

- **Tag a major version** (`v1`) pointing at the current commit, so consumers can use
  `uses: jysf/setup-crustyimg@v1`. Until that tag exists, reference `@main`.
- **List on the GitHub Marketplace** (optional) via the repo's *Releases → Publish this Action*.

## License

MIT OR Apache-2.0, matching crustyimg.
