# wrk

`wrk` is a simple, fast workspace managment CLI tool built on top of `tmux` and `fzf`.
It lets you find project, or project work session, with fuzzy search and jump straight into a persistent session for it.

Think of it as stripped-down version of [Conductor][1] that you can use in your terminal or over SSH.

## Dependencies

- [tmux][2]
- [fzf][3]
- OPTIONAL: [mise][4]

If `mise` is present, `wrk` runs `mise trust` on each new worktree so that `.env` variables (including `WRK_PORT`) load automatically via mise's env integration.

## Installation

You can install this package with Homebrew.

```sh
brew tap danhorst/taps && brew install wrk
```

OR

```sh
brew install danhorst/taps/wrk
```

If you don't want to use homebrew, you can just put `wrk` somewhere on your `$PATH` (e.g. `/usr/local/bin/wrk`) and make it executable:

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

### Multiple sessions per project

```sh
wrk webapp feature-branch
```

Attaches to (or creates) a session named `feature-branch` for the `webapp` project, leaving any other sessions for that project untouched.

When a project already has two or more sessions and no `session-name` is given, a second fuzzy finder appears listing the existing sessions plus a `[new]` option to create an additional named one.

### Worktree sessions via hotkey

In the project picker, press `ctrl-o` on any highlighted project to open a branch picker for it.
The branch picker lists all local and remote branches; selecting one opens (or resumes) a worktree-backed session for that branch.
Choose `[new]` to type a branch name — the worktree and branch are created automatically.

> [!NOTE]
> `ctrl-o` was chosen to avoid conflicts with the default tmux prefix (`ctrl-b`) and common readline bindings.
> If you have remapped your tmux prefix to `ctrl-o`, you will need to press it twice or use `wrk <project> <branch>` instead.

### Worktree Secrets Management

If the parent project has a `.env` file, it is copied into the new worktree (with any existing `WRK_PORT` entry stripped) so that project-level config carries over automatically.
This copy happens once, at worktree creation time.

> [!WARNING]
> `wrk` does not help you keep changes in your `.env` files in sync.
> You need to do this yourself.

### Worktree Port Configuration

Since `wrk` lets you spint up the same project in multiple workspaces, we need a way for more than one application server to run without binding to the same port.
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
- Kills the associated tmux session if one is running

### Inside an existing tmux session

`wrk` works the same way from inside tmux.
In an existing tmux session, it uses `switch-client` instead of `attach`, so you can jump between projects without leaving your current session.

## Configuration

| Variable            | Default            | Description                      |
| ------------------- | ------------------ | -------------------------------- |
| `WRK_PROJECT_ROOT`  | `~/git`            | Root directory scanned for repos |
| `WRK_WORKTREE_ROOT` | `~/.wrk/worktrees` | Root directory for worktrees     |

Repos are discovered at depth two: `$WRK_PROJECT_ROOT/<org>/<repo>/.git`.

## Session naming

| Project path                                 | Session                      |
| -------------------------------------------- | ---------------------------- |
| `~/git/acme/webapp`                          | `wrk-webapp`                 |
| `~/git/acme/webapp` + label `feature-branch` | `wrk-webapp__feature-branch` |
| `~/git/acme/my-app`                          | `wrk-my_app`                 |

Special characters in both the project name and session label are replaced with underscores.
The `__` (double underscore) separates the project name from the label, so sessions for different projects never collide.

[1]: https://www.conductor.build/
[2]: https://github.com/tmux/tmux
[3]: https://github.com/junegunn/fzf
[4]: https://mise.jdx.dev
