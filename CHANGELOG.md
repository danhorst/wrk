# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [1.5.0] - 2026-05-16

### Added
- Session indicators throughout all pickers and `--list`: ◆ = attached, ◇ = detached, blank = no session
- Project picker shows ◆/◇ per project based on live tmux sessions
- Branch picker (ctrl-o) shows ◆/◇ per branch based on existing worktree sessions
- Within-project session picker shows ◆/◇ per session
- `--list` STATUS column uses ◆/◇ instead of `active`/`—`; single upfront `tmux list-sessions` call replaces per-worktree `has-session` loop
- `--clean` pickers show ◆/◇ so attached sessions are visible before confirming removal
- `ctrl-x` hotkey in the project picker opens a picker of all live wrk sessions with ◆/◇ indicators and directory tree preview

## [1.4.0] - 2026-05-15

### Added
- `ctrl-t` hotkey in the project picker opens a named session for the highlighted project in its main directory — no worktree created; useful for long-running processes like dev servers
- Multi-session picker splits `[new]` into `[new session]` (same directory, no worktree) and `[new worktree]` (existing branch/worktree behavior)
- `--help` and README document the `ctrl-t` hotkey

## [1.3.2] - 2026-05-08

### Fixed
- fzf pickers no longer error when `FZF_DEFAULT_OPTS` sets a `bat` preview; project and worktree pickers now show a `tree` directory preview instead

### Added
- `tree` is now a required dependency

## [1.3.1] - 2026-05-05

### Fixed
- `mise trust` now runs when opening a main project directory, not only when creating worktrees

## [1.3.0] - 2026-04-30

### Added
- `ctrl-o` hotkey in the project picker opens a branch picker for the highlighted project, creating a worktree-backed session without typing the full project path
- Branch picker lists all local and remote branches (deduplicated); choose `[new]` to type a name and create a fresh branch and worktree in one step
- Project picker now shows a header hint (`enter: open  |  ctrl-o: open branch as worktree`) for discoverability
- `--help` documents the `ctrl-o` hotkey and the `--clean` branch/session side-effects
- README documents `mise` as an optional dependency, `WRK_PORT` assignment and `.env` inheritance, and the full `--clean` workflow

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

[Unreleased]: https://github.com/danhorst/wrk/compare/v1.5.0...HEAD
[1.5.0]: https://github.com/danhorst/wrk/compare/1.4.0...1.5.0
[1.4.0]: https://github.com/danhorst/wrk/compare/1.3.2...1.4.0
[1.3.2]: https://github.com/danhorst/wrk/compare/1.3.1...1.3.2
[1.3.1]: https://github.com/danhorst/wrk/compare/1.3.0...1.3.1
[1.3.0]: https://github.com/danhorst/wrk/compare/1.2.1...1.3.0
[1.2.1]: https://github.com/danhorst/wrk/compare/v1.2.0...v1.2.1
[1.2.0]: https://github.com/danhorst/wrk/compare/v1.1.0...v1.2.0
[1.1.0]: https://github.com/danhorst/wrk/compare/v1.0.0...v1.1.0
[1.0.0]: https://github.com/danhorst/wrk/releases/tag/v1.0.0
