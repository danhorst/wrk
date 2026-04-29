# wrk

Single-file bash script (`wrk`) that manages tmux sessions for git repos.

## Release process

Releases involve two repos: this one (`danhorst/wrk`) and the Homebrew tap
(`danhorst/homebrew-tap`, at `../homebrew-tap`).

Run the release script:
```sh
./scripts/release vX.Y.Z
```

The script handles CHANGELOG promotion, commit, annotated tag, push, SHA256 fetch, and formula update.
