# wrk

`wrk` is a simple, fast workspace managment CLI tool built on top of `tmux` and `fzf`.
It lets you find project, or project work session, with fuzzy search and jump straight into a persistent session for it.

Think of it as stripped-down version of [Conductor][1] manifsted as a shell script.

## Dependencies

- [tmux][2]
- [fzf][3]

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
wrk webapp tests
```

Attaches to (or creates) a session named `tests` for the `webapp` project, leaving any other sessions for that project untouched.

When a project already has two or more sessions and no `session-name` is given, a second fuzzy finder appears listing the existing sessions plus a `[new]` option to create an additional named one.

### Inside an existing tmux session

`wrk` works the same way from inside tmux — it uses `switch-client` instead of `attach`, so you can jump between projects without leaving your current session.

## Configuration

| Variable           | Default | Description                      |
| ------------------ | ------- | -------------------------------- |
| `WRK_PROJECT_ROOT` | `~/git` | Root directory scanned for repos |

Repos are discovered at depth two: `$WRK_PROJECT_ROOT/<org>/<repo>/.git`.

## Session naming

| Project path                        | Session              |
| ----------------------------------- | -------------------- |
| `~/git/acme/webapp`                 | `wrk-webapp`        |
| `~/git/acme/webapp` + label `tests` | `wrk-webapp__tests` |
| `~/git/acme/my-app`                 | `wrk-my_app`        |

Special characters in both the project name and session label are replaced with underscores.
The `__` (double underscore) separates the project name from the label, so sessions for different projects never collide.

[1]: https://www.conductor.build/
[2]: https://github.com/tmux/tmux
[3]: https://github.com/junegunn/fzf
