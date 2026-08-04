# wrk

Single-file bash script (`wrk`) that manages tmux sessions for git repos.

## Testing an unreleased build

```sh
./scripts/pre-release on      # `wrk` resolves to this working tree
./scripts/pre-release off     # back to the Homebrew build
./scripts/pre-release status  # which build is live
./scripts/pre-release         # toggle
```

Symlinks the repo copy into `~/.local/bin` (override with `WRK_SHADOW_DIR`) and
unlinks the Homebrew keg when its bin precedes that directory on PATH. The keg
stays installed, so `off` restores it with `brew link`.

The symlink points at the file, so edits are live — and a `git checkout` here
changes `wrk` system-wide until it's switched off. Turn it off before `brew
upgrade wrk`, or the upgrade won't take effect.

## Release process

Releases involve two repos: this one (`danhorst/wrk`) and the Homebrew tap
(`danhorst/homebrew-tap`, at `../homebrew-tap`).

Run the release script:
```sh
./scripts/release vX.Y.Z
```

The script handles CHANGELOG promotion, commit, annotated tag, push, SHA256 fetch, and formula update.

Before running: populate `## [Unreleased]` in CHANGELOG.md with the release notes.

The release commit contains only `wrk` and `CHANGELOG.md` — commit README or any other changes separately before running the script.
