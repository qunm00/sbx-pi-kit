# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- `PI_SBX_SESSIONS` env var: host directory for persistent pi sessions, shared by
  every project. Mounted read-write automatically (unless already covered by
  `PI_SBX_MOUNTS`) and used as `PI_CODING_AGENT_SESSION_DIR`. Default `~/.pi-sbx`,
  sessions at `<dir>/sessions`
- `~` is now expanded in `PI_SBX_MOUNTS` and `PI_SBX_SESSIONS` entries
- `PI_SBX_MOUNTS` env var: comma-separated absolute host paths mounted into the
  sandbox at the same path, `:ro` for read-only. The resolved list is recorded in
  `<PI_SBX_SESSIONS>/sbx-mounts`; a changed list warns and suggests `--new`
  (sbx fixes mounts at create time)
- `--name <name>` flag to override the default `pi-sbx` sandbox name
- `PI_SBX_DRY_RUN=1` to print the resolved `sbx` commands instead of running them

### Changed
- pi now starts in the directory `pi-sbx` was launched from instead of the
  workdir the sandbox was created with, so the shared `pi-sbx` sandbox follows
  the project you are in. When that directory is not mounted (`PI_SBX_MOUNTS`
  is set), pi starts in the home directory
- `PI_SBX_MOUNTS` now replaces the workspace mount instead of adding to it: when
  it is set, the current directory (and the `dir` positional) is not mounted. When
  it is unset, the workspace directory is mounted as before
- The sandbox name is now the fixed `pi-sbx` for every project, so one sandbox
  serves all projects. The `pi-<dirname>` default is removed; use `--name` to change it
- pi sessions are stored in `PI_SBX_SESSIONS` (default `~/.pi-sbx`) instead of
  `<workspace>/.pi/sessions`, so they are shared by all projects
- **Deprecated**: `--kit` is no longer passed to `sbx`; the kit ref is now the first
  positional argument of `sbx create` / `sbx run`
- `pi-sbx` now accepts the kit as its first positional argument (`pi-sbx <kit> [dir]`)
- **Removed**: the `--kit` flag (it now errors, telling you to use the positional form)

## [1.0.1] - 2026-07-13

### Changed
- Migrated from `kind: agent` to `kind: sandbox` and removed entry point based start up

### Added
- `pi-sbx` launcher script: `sbx create` + `sbx exec pi` lifecycle
- `--new` flag for fresh sandbox creation
- `--version` flag for checking pi version in running sandbox
- Bashrc auto-launch of `pi` in interactive shell sessions
- `pi-sbx` hardcodes `KIT_DIR` git URL — standalone script, no local checkout needed
- `--kit` flag to override kit path/URL

## [1.0.0] - 2026-07-11

### Added
- Initial release
- OpenRouter free-tier support
- GitHub Copilot integration
- Node 24 LTS base image
- Shell alias for quick startup