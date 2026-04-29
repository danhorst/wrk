# wrk

A minimal tmux session manager for git repositories. Pick a project with fuzzy
search and jump straight into a persistent session for it.

## Dependencies

- [tmux](https://github.com/tmux/tmux)
- [fzf](https://github.com/junegunn/fzf)

## Installation

Put `wrk` somewhere on your `$PATH` (e.g. `/usr/local/bin/wrk`) and make it executable:

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

Opens a fuzzy finder over all git repositories under `~/git`. Selecting a project
attaches to an existing tmux session for it, or creates one if none exists.

```sh
wrk webapp
```

Same, but pre-filters the list to repos matching `webapp`.

### Multiple sessions per project

```sh
wrk webapp tests
```

Attaches to (or creates) a session named `tests` for the `webapp` project,
leaving any other sessions for that project untouched.

When a project already has two or more sessions and no `session-name` is given,
a second fuzzy finder appears listing the existing sessions plus a `[new]` option
to create an additional named one.

### Inside an existing tmux session

`wrk` works the same way from inside tmux — it uses `switch-client` instead of
`attach`, so you can jump between projects without leaving your current session.

## Configuration

| Variable | Default | Description |
|---|---|---|
| `WRK_PROJECT_ROOT` | `~/git` | Root directory scanned for repos |

Repos are discovered at depth two: `$WRK_PROJECT_ROOT/<org>/<repo>/.git`.

## Session naming

| Project path | Session |
|---|---|
| `~/git/acme/webapp` | `proj-webapp` |
| `~/git/acme/webapp` + label `tests` | `proj-webapp__tests` |
| `~/git/acme/my-app` | `proj-my_app` |

Special characters in both the project name and session label are replaced with
underscores. The `__` (double underscore) separates the project name from the
label, so sessions for different projects never collide.
