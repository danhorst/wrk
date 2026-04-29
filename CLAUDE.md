# wrk

Single-file bash script (`wrk`) that manages tmux sessions for git repos.

## Release process

Releases involve two repos: this one (`danhorst/wrk`) and the Homebrew tap
(`danhorst/homebrew-wrk`, at `../homebrew-wrk`).

Steps for a new release:

1. Update `CHANGELOG.md` — move items from `[Unreleased]` to a new `[X.Y.Z]` section with today's date, and update the comparison links at the bottom.
2. Commit all changes to `wrk`, `README.md`, and `CHANGELOG.md`.
3. Tag with an **annotated** tag (plain `git tag` will fail on push):
   ```sh
   git tag -a vX.Y.Z -m "vX.Y.Z"
   git push origin main && git push origin vX.Y.Z
   ```
4. Fetch the SHA256 of the new GitHub tarball:
   ```sh
   curl -sL https://github.com/danhorst/wrk/archive/refs/tags/vX.Y.Z.tar.gz | shasum -a 256
   ```
5. Update `../homebrew-wrk/Formula/wrk.rb` — change `url` and `sha256`.
6. Commit and push the formula:
   ```sh
   cd ../homebrew-wrk
   git add Formula/wrk.rb && git commit -m "Update wrk to vX.Y.Z"
   git push origin main
   ```
