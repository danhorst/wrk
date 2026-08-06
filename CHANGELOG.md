# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [2.1.0] - 2026-08-06

### Added
- `ctrl-d` in either session picker terminates the highlighted session after a confirmation prompt. Both pickers now loop rather than exiting, so the list re-primes after each kill and several sessions can be cleared in one pass. Under `aoe` this is a plain `aoe remove`: the record goes to the trash and `aoe session restore` brings it back, while the worktree and branch are left for `--clean`. Without `aoe`, the tmux session is killed outright. Until now the only teardown path was `--clean`, which could not reach a session whose worktree you wanted to keep

### Fixed
- `--clean` now purges trashed session records rooted in a worktree it removes, not just live ones. `aoe_purge_worktree` read the same row cache the pickers do, which has the trash subtracted, so a session ended with `ctrl-d` survived the removal of the directory it named. Nothing else collected it: aoe's retention sweep runs only inside the `aoe serve` daemon, and wrk filtered the record out of every picker for the same reason the purge missed it. The unfiltered rows are now kept alongside the filtered ones and the purge reads those
- `wrk` no longer prints `awk: newline in string` on every invocation when aoe's trash holds two or more sessions. The trashed ids were passed to awk through `-v` one per line, and the awk macOS ships warns on each embedded newline in a `-v` value; they are now space-separated, which ids cannot contain. Cosmetic, but `ctrl-d` puts a session in the trash every time it is used, so the noise would have become permanent

## [2.0.0] - 2026-08-04

### Added
- Interoperability with [Agent of Empires](https://github.com/agent-of-empires/agent-of-empires) (`aoe`). When `aoe` is on `$PATH` it owns session lifecycle: wrk still discovers projects, picks branches, and provisions worktrees, then calls `aoe add` instead of `tmux new-session`, so sessions appear in the aoe TUI, web dashboard, and status polling. Worktree sessions launch `claude`; project and vault sessions still open a shell. The `<org>` directory becomes the aoe group and `<repo>`/`<repo>-<branch>` the session title
- `jq` is now a dependency, used to read `aoe list --json`
- `WRK_AOE_SHELL_TOOL` names the `aoe` custom agent used for shell sessions (default `shell`). `aoe` validates `--cmd` against its own agent allowlist and rejects a bare shell, so non-worktree sessions run through a custom agent declared in its config; `wrk` exports `SSH_AUTH_SOCK` before handing off so a bare `environment = ["SSH_AUTH_SOCK"]` passthrough forwards the agent symlink without hardcoding a path
- On every attach, wrk sets `allow-passthrough` on the aoe session and narrows `remain-on-exit`. `aoe` leaves `allow-passthrough` alone for anyone who has a `~/.tmux.conf`, on the assumption they manage it themselves, which broke key and escape-sequence handling in `claude` sessions. Its `remain-on-exit on` left a dead, unclosable pane behind every session that ended normally; wrk now sets `failed` for agent sessions, so the pane survives only a non-zero exit and a crashed run stays readable, and `off` for shell sessions, where a bare `exit` reports the last command's status and would tombstone at random. Both options are re-applied on each attach because `aoe` rebuilds them whenever it restarts a stopped session
- Live status indicators in every picker and `--list`: `✖` error, `●` waiting on you, `◆` attached, `◐` working, `◇` idle or stopped. A project row shows the most urgent indicator among its own sessions and its worktrees', so a branch waiting on input outranks an attached session elsewhere in the project. The three agent states come from the status file `aoe`'s hooks write, which the agent itself updates, so they stay current whether or not the aoe TUI or web dashboard is running. Without `aoe`, rows are `◆`, `◇`, or blank as before

### Changed
- **BREAKING** — under `aoe`, sessions are identified by path rather than by name, so the `wrk-<repo>__<label>` scheme does not apply and existing `wrk-*` sessions are not adopted. Worktrees created by earlier versions stay on disk and keep working, but have no aoe session record; kill and recreate the sessions once after upgrading. Without `aoe` installed, behavior is unchanged
- wrk keeps ownership of the worktrees it creates. They stay at `$WRK_WORKTREE_ROOT/<org>/<repo>/<branch>` and aoe adopts them as `managed_by_aoe: false`, so aoe never moves or deletes one — which also preserves the `<org>` path component that aoe's own `worktree.path_template` has no token for
- `--clean` purges the aoe session record before removing a worktree, so its `on_destroy` hooks still see a live checkout and no orphan row is left behind
- Session handling is routed through one of two interchangeable backends — `tmux` or `aoe` — chosen once at startup instead of re-decided at each call site. The pickers, indicators, and cleanup paths existed in two parallel copies, one per mode, which is now a single implementation over a shared row format

### Performance
- Project picker startup drops to roughly half its previous cost with `aoe` installed, and to a third without it, by resolving each row through in-process parameter expansion rather than a command substitution per project, and by fetching the session list once per run instead of once per indicator

### Fixed
- `allow-passthrough` now actually reaches sessions wrk creates itself, which Claude Code needs for key and escape-sequence handling. Two separate faults hid it: from inside tmux the option was set against a session target, but `allow-passthrough` is a window option, so tmux answered "no such window" and the trailing `|| true` reported success (since 1.6.3, when exact `=` targeting was introduced); from outside tmux it was set after a blocking `tmux new-session`, so it only ran once the session was detached again (since 1.6.0). Sessions are now created detached, given the option against the window, and attached afterwards. It lands on the session's first window, so a window opened later still needs its own
- Under `aoe`, a session trashed in the TUI is no longer resurrected when you reopen its project. `aoe list` reports trashed records and its JSON carries no field to tell them apart, so wrk now subtracts the trash explicitly
- Under `aoe`, a worktree session is matched to its project through aoe's own record of the repo it was cut from, rather than by testing whether its path sits under `$WRK_WORKTREE_ROOT/<org>/<repo>/`. The old test silently stopped grouping if either tool's layout changed
- The branch picker no longer shows another session's indicator against a branch. It matched on the worktree directory alone, so a `ctrl-t` session sharing that directory shadowed the branch's own state
- `--clean` now classifies merged worktrees correctly. `git branch --merged` marks a branch that is checked out in another worktree with `+`, which every wrk worktree branch is by definition, so the whitespace-only prefix the match expected never lined up and nothing was ever pre-selected for removal. Branches are now listed with `--format='%(refname:short)'` and matched exactly, which also stops a `.` in a branch name from acting as a wildcard

## [1.7.1] - 2026-07-18

### Fixed
- Sessions no longer carry a stale SSH agent socket after reattaching from a different SSH connection. `SSH_AUTH_SOCK` is now exported as a stable symlink (`ssh-auth-sock`, alongside `worktrees` under the wrk root) that wrk repoints to the current agent on every invocation, so new sessions and new panes in existing sessions always resolve to the live agent. Pre-existing sessions still carry their original socket path and need to be recreated once to pick this up

## [1.7.0] - 2026-06-28

### Added
- PKM vault sessions. Flag a non-git directory as a vault with a marker file (default `.wrk-vault`) and point `WRK_VAULT_ROOTS` at the vault or a folder of vaults; vaults appear in the picker tagged with 📓 and open a plain session (no worktree, branch, or `WRK_PORT`). Marker filename is configurable via `WRK_VAULT_MARKER`

## [1.6.3] - 2026-06-09

### Fixed
- Selecting `[default]` in the sub-picker now creates a fresh default session instead of attaching to an existing labeled session for the same project. tmux's target syntax falls back to prefix matching, so `wrk-foo` was resolving to an existing `wrk-foo__bar`; targets are now anchored with `=` for exact-name matching

## [1.6.2] - 2026-06-06

### Fixed
- Default session is now matched correctly in the multi-session picker. The previous regex was anchored to end-of-line and didn't account for the trailing tab/attached-count field, so the bare `$BASE` session never appeared in `_sess_info` — leaving `[default]` without its ◆/◇ indicator and underlying the named-session-not-viewable issue addressed cosmetically in v1.6.1
- `[default]` is now always emitted as the first entry in the sub-picker with its real indicator, and the default session is no longer duplicated under its raw `wrk-…` name

## [1.6.1] - 2026-06-06

### Fixed
- Session picker now appears when a project has only named (non-worktree) sessions and no default session; previously the default session was opened unconditionally, hiding the named ones
- Projects with only labeled sessions now include a `[default]` option in the session picker, making the unlabeled base session reachable without exiting the picker

## [1.6.0] - 2026-06-05

### Added

- Set `allow-passthrough on` on all new tmux sessions so cmux notification rings and tab badges work out of the box when running Claude Code or other OSC-aware agents inside `wrk` sessions.
  Degrades gracefully on tmux < 3.3.

## [1.5.4] - 2026-05-28

### Fixed
- New worktree sessions open in the worktree directory on first creation; `git worktree add` stdout was being captured by the command substitution that resolves the session directory, producing an invalid path and causing tmux to fall back to the working directory where `wrk` was invoked

## [1.5.3] - 2026-05-18

### Performance
- Project picker startup drops from ~6s to ~0.1s on a 65-repo machine by replacing `tr`/`sed`/`grep`/`dirname`/`basename` pipelines with bash built-in parameter expansion and an in-process array walk (~650 fewer subprocess forks)

## [1.5.2] - 2026-05-17

### Fixed
- `--clean` now fetches the base branch from origin before checking merge status, preventing false positives when local refs are stale
- Branch picker (`ctrl-o`) prunes deleted remote branches before listing, keeping stale entries out of the picker

### Changed
- Release workflow supports manual trigger (`workflow_dispatch`) for re-running or testing without a new tag push

## [1.5.1] - 2026-05-16

### Fixed
- `ctrl-x` with no active sessions now shows a status-bar message and returns to the project picker instead of silently exiting

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

[Unreleased]: https://github.com/danhorst/wrk/compare/v2.1.0...HEAD
[2.1.0]: https://github.com/danhorst/wrk/compare/2.0.0...2.1.0
[2.0.0]: https://github.com/danhorst/wrk/compare/1.7.1...2.0.0
[1.7.1]: https://github.com/danhorst/wrk/compare/1.7.0...1.7.1
[1.7.0]: https://github.com/danhorst/wrk/compare/1.6.3...1.7.0
[1.6.3]: https://github.com/danhorst/wrk/compare/1.6.2...1.6.3
[1.6.2]: https://github.com/danhorst/wrk/compare/1.6.1...1.6.2
[1.6.1]: https://github.com/danhorst/wrk/compare/1.6.0...1.6.1
[1.6.0]: https://github.com/danhorst/wrk/compare/1.5.4...1.6.0
[1.5.4]: https://github.com/danhorst/wrk/compare/1.5.3...1.5.4
[1.5.3]: https://github.com/danhorst/wrk/compare/1.5.2...1.5.3
[1.5.2]: https://github.com/danhorst/wrk/compare/1.5.1...1.5.2
[1.5.1]: https://github.com/danhorst/wrk/compare/v1.5.0...v1.5.1
[1.5.0]: https://github.com/danhorst/wrk/compare/1.4.0...1.5.0
[1.4.0]: https://github.com/danhorst/wrk/compare/1.3.2...1.4.0
[1.3.2]: https://github.com/danhorst/wrk/compare/1.3.1...1.3.2
[1.3.1]: https://github.com/danhorst/wrk/compare/1.3.0...1.3.1
[1.3.0]: https://github.com/danhorst/wrk/compare/1.2.1...1.3.0
[1.2.1]: https://github.com/danhorst/wrk/compare/v1.2.0...v1.2.1
[1.2.0]: https://github.com/danhorst/wrk/compare/v1.1.0...v1.2.0
[1.1.0]: https://github.com/danhorst/wrk/compare/v1.0.0...v1.1.0
[1.0.0]: https://github.com/danhorst/wrk/releases/tag/v1.0.0
