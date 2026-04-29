# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [1.2.1] - 2026-04-29

### Added
- `--version` flag reports the current version

## [1.2.0] - 2026-04-29

### Added
- Git worktree management for labeled sessions: each label gets an isolated checkout in `WRK_WORKTREE_ROOT` (default: `~/.wrk/worktrees`)
- Branch resolution order: existing local branch → existing remote branch (after `git fetch`) → new branch from HEAD
- `WRK_WORKTREE_ROOT` environment variable to configure worktree storage location
- `--list` subcommand: tabular view of all worktrees with branch, port, and tmux session status
- `--clean` subcommand: fzf-based review and removal of worktrees; merged branches pre-selected, unmerged offered in a second pass; prunes stale git worktree metadata before presenting
- `--help` subcommand
- `WRK_PORT` assignment: each new worktree receives a unique, non-conflicting port written to its `.env`
- `.env` copied from parent repo into each new worktree (with `WRK_PORT` stripped so a fresh value is always assigned)
- `mise trust` run on new worktrees so mise loads the worktree `.env` automatically; warns if mise is not installed

### Changed
- Branch names now preserve hyphens; only tmux session suffixes are restricted to alphanumerics and underscores
- `set -euo pipefail` for stricter error handling

## [1.1.0] - 2026-04-28

### Added
- Multi-session support: pass a session label as a second argument (`wrk [filter] [session-name]`)
- Auto session picker via a second fzf when a project has two or more active sessions
- Works correctly from inside an existing tmux session (`switch-client` instead of `attach`)
- README with usage instructions and configuration reference

## [1.0.0] - 2026-04-06

### Added
- Initial release: fuzzy project picker backed by fzf, one tmux session per git repo

[Unreleased]: https://github.com/danhorst/wrk/compare/v1.2.1...HEAD
[1.2.1]: https://github.com/danhorst/wrk/compare/v1.2.0...v1.2.1
[1.2.0]: https://github.com/danhorst/wrk/compare/v1.1.0...v1.2.0
[1.1.0]: https://github.com/danhorst/wrk/compare/v1.0.0...v1.1.0
[1.0.0]: https://github.com/danhorst/wrk/releases/tag/v1.0.0
