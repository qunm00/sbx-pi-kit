## Learned User Preferences

- Keep sandbox-isolated user namespace (`~/.pi/agent/`) separate from synced project namespace (`<workspace>/.pi/`)
- Sync only sessions into the project folder, not full config with auth tokens
- Manage credentials through sbx secrets (env vars) rather than pi's auth.json
- Use `PI_CODING_AGENT_SESSION_DIR` to redirect session storage to the mounted workspace
- Use `.pi/settings.json` for project settings that persist across sandbox instances
- Prefer clean breaks over backward-compatibility shims: remove deprecated flags/paths outright instead of warning
- Delegate memory/analysis work to a dedicated subagent (e.g. `continual-learning-updater`) rather than running it inline
- Prefer concise reports (roughly under 150 words) that state exact command forms
- Does not use project-level skills; personal skills come from a git-installed skill repo

## Learned Workspace Facts

- Project `sbx-pi-kit` is a Docker Sandbox kit for running the `pi` coding agent; image is `nmiquan/sbx-pi-template:latest` (base `docker/sandbox-templates:shell`, runs as non-root `agent`)
- Workspace is bind-mounted at its absolute host path inside the sandbox
- Launcher script `pi-sbx` creates per-directory sandboxes (`pi-<dirname>`); usage is `pi-sbx [kit] [dir]`
- The sandbox kit is passed as the first positional argument (`sbx create <kit> --name foo pi <dir>`); `--kit` is reserved for mixin kits, and `pi-sbx --kit` is a hard error
- Key sbx commands: `sbx ls`, `sbx create`, `sbx run`, `sbx exec`, `sbx rm`; kit args via `--kit-arg` / `--kit-args-file`
- Sandboxes are ephemeral per-directory: only the image and the bind-mounted workspace persist, and in-sandbox installs like `pi update --extensions` are lost on recreate
- Durable toolchain belongs in the image; project-specific tools belong in a workspace-mounted bootstrap script
- Personal skills come from `https://github.com/qunm00/pi-skills`, git-installed under `~/.pi/agent/git/.../skills/`
- Packages installed via `~/.pi/agent/settings.json`: `git:github.com/qunm00/pi-continual-learning` and `npm:@tintinweb/pi-subagents`
- pi-subagents discovers agent types only from `.pi/agents/`, `.agents/agents/`, and `~/.pi/agent/agents/`
- `.pi/` holds `state/` and `sessions/`; `.pi/skills/` was emptied of project skills
- Package source: `git+https://github.com/qunm00/sbx-pi-kit.git`
- Template source: `https://github.com/qunm00/sbx-pi-template`
