# wrk

`wrk` is a simple, fast workspace managment CLI tool built on top of `tmux` and `fzf`.
It lets you find project, or project work session, with fuzzy search and jump straight into a persistent session for it.

Think of it as stripped-down version of [Conductor][1] that you can use in your terminal or over SSH.

## Dependencies

- [fzf][2]
- [tmux][3]
- [tree][4]
- [jq][8]
- OPTIONAL: [mise][5]
- OPTIONAL: [Agent of Empires][9]

If `mise` is present, `wrk` runs `mise trust` on each new [worktree][6] so that `.env` variables (including `WRK_PORT`) load automatically via mise's env integration.

If `aoe` is present, it takes over session lifecycle — see [Agent of Empires](#agent-of-empires).

## Installation

You can install this package with [Homebrew][7].

```sh
brew tap danhorst/tap && brew install wrk
```

OR

```sh
brew install danhorst/tap/wrk
```

If you don't want to use Homebrew, you can just put `wrk` somewhere on your `$PATH` (e.g. `/usr/local/bin/wrk`) and make it executable:

```sh
chmod +x wrk
```

## Usage

```
wrk [filter] [session-name]
```

### Basic

```sh
wrk
```

Opens a fuzzy finder over all git repositories under `WRK_PROJECT_ROOT` which defaults to `~/git`.
Selecting a project attaches to an existing tmux session for it, or creates one if none exists.

```sh
wrk webapp
```

Opens the same fuzzy search but pre-filters the list to repos matching `webapp`.

### Session indicators

Every picker, and `--list`, marks each row with the state of its session.

| Indicator | Meaning                                |
| --------- | -------------------------------------- |
| `✖`       | Agent hit an error                     |
| `●`       | Agent is waiting on you                |
| `◆`       | Session is currently attached          |
| `◐`       | Agent is working                       |
| `◇`       | Session exists but is idle or stopped  |
| (none)    | No session                             |

A project row shows the most urgent indicator among its own sessions and those of its worktrees, in the order listed above.
A branch waiting on input therefore outranks an attached session elsewhere in the same project, which is what makes the project picker usable as a queue.

`✖`, `●` and `◐` require [Agent of Empires](#agent-of-empires).
They are read from the status file `aoe`'s hooks write, which the agent itself updates — so they stay current whether or not the `aoe` TUI or its web dashboard is running.
Without `aoe`, rows are only ever `◆`, `◇`, or blank.

### Multiple sessions per project

```sh
wrk webapp feature-branch
```

Attaches to (or creates) a session named `feature-branch` for the `webapp` project.

When a project has two or more sessions and no `session-name` is given, a second fuzzy finder list is presented containing the existing sessions along with the `[new session]` and `[new worktree]` options.

### Named sessions via hotkey

In the project picker, press `ctrl-t` on any highlighted project to open a named session for it in the same directory.
A “session” _does not_ create a dedicated worktree for the session.
It is useful for a long-running process like a dev server that should share the working directory with your main session.

### Worktree sessions via hotkey

In the project picker, press `ctrl-o` on any highlighted project to open a branch picker for it.
The branch picker lists all local and remote branches; selecting one opens (or resumes) a worktree-backed session for that branch.
Choose `[new]` to type a branch name — the worktree and branch are created automatically.

> [!NOTE]
> `ctrl-o` was chosen to avoid conflicts with the default tmux prefix (`ctrl-b`) and common readline bindings.
> If you have remapped your tmux prefix to `ctrl-o`, you will need to press it twice or use `wrk <project> <branch>` instead.

### Terminating sessions via hotkey

`ctrl-x` in the project picker switches to a picker of every live session; `ctrl-d` there terminates the highlighted one after a confirmation prompt.
The same key works in the per-project session picker you get when a project has more than one session.
The picker stays open afterwards, so several sessions can be cleared in a row.

Terminating only ends the session.
The worktree and its branch are left on disk for `wrk --clean` to deal with.
Under `aoe` the session record moves to aoe's trash and can be brought back with `aoe session restore <id>`; without `aoe`, the tmux session is killed outright and there is nothing to restore.

### Worktree Secrets Management

If the parent project has a `.env` file, it is copied into the new worktree (with any existing `WRK_PORT` entry stripped) so that project-level config carries over automatically.
This copy happens once, at worktree creation time.

> [!WARNING]
> `wrk` does not help you keep changes in your `.env` files in sync.
> You need to do it yourself.

### Worktree Port Configuration

Since `wrk` lets you spin up the same project in multiple workspaces, we need a way for more than one application server to run without binding to the same port.
To address this, `wrk` adds a unique `WRK_PORT` to the `.env` file in a worktree when it is created.
If `WRK_PORT` is already set in the parent project, that value will be stripped out when the `.env` file is copied over.
The value for `WRK_PORT` is guaranteed not to collide with any other worktree's port and not to be already bound on the host.

Use `WRK_PORT` directly in your app config, or map it to a framework-specific variable in the project's `.env`:

```sh
PORT=$WRK_PORT          # Puma, Rack, many others
CONDUCTOR_PORT=$WRK_PORT
```

> [!NOTE]
> Without `mise`, `WRK_PORT` is written to `.env` but won't load into the shell automatically.
> `wrk` will warn you about this and you will need to source the file yourself or use a different env loader.

### Cleaning up worktrees (`--clean`)

`wrk --clean` presents two interactive fzf prompts:

1. **Merged worktrees** — branches already merged into the default branch on `origin` are selected by default. Deselect any you want to keep, then confirm.
2. **Unmerged worktrees** — listed separately with nothing pre-selected, so you have to opt in explicitly.

For each removed worktree, `wrk` also:

- Deletes the local git branch (using `git branch -d`; skipped with a warning if the branch has unmerged commits)
- Kills the associated tmux session if one is running, or purges the `aoe` session record when running under `aoe`

### PKM vault sessions

`wrk` can also manage sessions for personal-knowledge-management workspaces — Obsidian vaults or OKF Knowledge Bundles — that are _not_ git-backed and live anywhere on disk.

Flag a directory as a vault by creating a marker file at its root:

```sh
touch ~/notes/.wrk-vault
```

Then point `WRK_VAULT_ROOTS` at the vault, or at a folder of vaults:

```sh
export WRK_VAULT_ROOTS="$HOME/notes"          # a single vault
export WRK_VAULT_ROOTS="$HOME/vaults"         # a folder scanned one level deep
export WRK_VAULT_ROOTS="$HOME/notes:$HOME/vaults"  # several, colon-separated
```

Each entry is either a vault itself (it contains the marker) or a folder whose immediate subdirectories are checked for the marker.
Vaults appear in the main picker tagged with 📓.
Selecting one opens (or resumes) a plain session rooted at the vault — there is no worktree, branch, or `WRK_PORT`, since vaults aren't git repos.
`ctrl-t` still opens named sub-sessions (`wrk-vault-<name>__<label>`); `ctrl-o` has no effect on a vault.

The marker filename defaults to `.wrk-vault` and can be changed with `WRK_VAULT_MARKER`.

### Agent of Empires

[Agent of Empires][9] (`aoe`) is a TUI and web dashboard for AI coding agents, also built on tmux.
When `aoe` is on your `$PATH`, `wrk` hands session lifecycle to it: `wrk` still discovers projects, picks branches, and provisions worktrees, then calls `aoe add` instead of `tmux new-session`.
Sessions created this way show up in the `aoe` TUI, its web dashboard, and its status polling.

Worktree sessions launch `claude`.
Plain project sessions and vault sessions open a shell, as they always have.

The `<org>` directory becomes the `aoe` group and `<repo>` or `<repo>-<branch>` becomes the session title, so `~/git/acme/webapp` on branch `fix-login` lands in group `acme` titled `webapp-fix-login`.

`wrk` keeps ownership of the worktrees it creates.
They stay at `$WRK_WORKTREE_ROOT/<org>/<repo>/<branch>`, `aoe` adopts them as unmanaged (`managed_by_aoe: false`), and it will never move or delete one.
Do not set `aoe`'s `worktree.path_template` — its templates have no `<org>` component, so letting `aoe` place worktrees would make `acme/webapp` and `other/webapp` collide.

Sessions are matched by path rather than by name, so the [session naming](#session-naming) scheme below does not apply under `aoe`.
A worktree session is tied to its project through `aoe`'s own record of the repo it was cut from, not by where the directory sits, so the two tools can disagree about layout without the grouping breaking.

Sessions you trashed in the `aoe` TUI are excluded, so opening the project creates a fresh session rather than resurrecting the discarded one.

Two entries are required in `~/.config/agent-of-empires/config.toml` (or `~/.agent-of-empires/config.toml` if you don't use XDG):

```toml
environment = ["SSH_AUTH_SOCK"]

[session]
custom_agents = { shell = "zsh" }
```

`aoe` refuses to launch anything outside its agent allowlist, so shell sessions run through a custom agent.
Name it something else and point `WRK_AOE_SHELL_TOOL` at that name.

The `environment` entry forwards the SSH agent symlink, so sessions keep working when you reattach over a different SSH connection.
`wrk` exports `SSH_AUTH_SOCK` pointing at the symlink before handing off, so a bare passthrough is enough — no absolute path, which matters because `aoe` does not expand `~`.

> [!NOTE]
> Leave `aoe`'s `tmux.socket_name` unset.
> Setting it moves `aoe`'s sessions to a private tmux server, where `wrk` can no longer see them or switch between them and its own.

### Inside an existing tmux session

`wrk` works the same way from inside tmux.
In an existing tmux session, it uses `switch-client` instead of `attach`, so you can jump between projects without leaving your current session.

## Configuration

| Variable            | Default            | Description                      |
| ------------------- | ------------------ | -------------------------------- |
| `WRK_PROJECT_ROOT`  | `~/git`            | Root directory scanned for repos |
| `WRK_WORKTREE_ROOT` | `~/.wrk/worktrees` | Root directory for worktrees     |
| `WRK_VAULT_ROOTS`   | _(unset)_          | Colon-separated vaults or folders of vaults |
| `WRK_VAULT_MARKER`  | `.wrk-vault`       | Filename that flags a vault root |
| `WRK_AOE_SHELL_TOOL` | `shell`           | Name of the `aoe` custom agent used for shell sessions |

Repos are discovered at depth two: `$WRK_PROJECT_ROOT/<org>/<repo>/.git`.
Vaults are discovered from `WRK_VAULT_ROOTS`: each entry is either a vault (it contains the marker file) or a folder scanned one level deep for vaults.

## Session naming

This applies when `aoe` is not installed.
Under `aoe`, sessions are identified by path and named by `aoe` — see [Agent of Empires](#agent-of-empires).

| Project path                                 | Session                      |
| -------------------------------------------- | ---------------------------- |
| `~/git/acme/webapp`                          | `wrk-webapp`                 |
| `~/git/acme/webapp` + label `feature-branch` | `wrk-webapp__feature-branch` |
| `~/git/acme/my-app`                          | `wrk-my_app`                 |
| `~/notes` (vault)                            | `wrk-vault-notes`            |
| `~/notes` (vault) + label `daily`            | `wrk-vault-notes__daily`     |

Special characters in both the project name and session label are replaced with underscores.
The `__` (double underscore) separates the project name from the label, so sessions for different projects never collide.
Vault sessions use a `wrk-vault-` prefix so they never collide with a git repo of the same name.

[1]: https://www.conductor.build/
[2]: https://github.com/junegunn/fzf
[3]: https://github.com/tmux/tmux
[4]: https://github.com/Old-Man-Programmer/tree
[5]: https://mise.jdx.dev
[6]: https://git-scm.com/docs/git-worktree
[7]: https://brew.sh
[8]: https://jqlang.org
[9]: https://github.com/agent-of-empires/agent-of-empires
