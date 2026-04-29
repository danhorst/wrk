# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [1.1.0] - 2026-04-28

### Added
- Multi-session support: pass a session label as a second argument (`wrk [filter] [session-name]`)
- Auto session picker via a second fzf when a project has two or more active sessions
- Works correctly from inside an existing tmux session (`switch-client` instead of `attach`)
- README with usage instructions and configuration reference

## [1.0.0] - 2026-04-06

### Added
- Initial release: fuzzy project picker backed by fzf, one tmux session per git repo

[Unreleased]: https://github.com/danhorst/wrk/compare/v1.1.0...HEAD
[1.1.0]: https://github.com/danhorst/wrk/compare/v1.0.0...v1.1.0
[1.0.0]: https://github.com/danhorst/wrk/releases/tag/v1.0.0
