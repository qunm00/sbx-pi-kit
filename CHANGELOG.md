# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

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