## Learned User Preferences

- Keep sandbox-isolated user namespace (`~/.pi/agent/`) separate from synced project namespace (`<workspace>/.pi/`)
- Sync only sessions into the project folder, not full config with auth tokens
- Manage credentials through sbx secrets (env vars) rather than pi's auth.json
- Use `PI_CODING_AGENT_SESSION_DIR` to redirect session storage to the mounted workspace
- Use `.pi/settings.json` for project settings that persist across sandbox instances
- Prefer clean breaks over backward-compatibility shims: remove deprecated flags/paths outright instead of warning
- Delegate memory/analysis work to a dedicated subagent (e.g. `continual-learning-updater`) rather than running it inline
- Prefer concise reports (roughly under 150 words) that state exact command forms
- Keep the sandbox image for toolchain only; no skills baked into the image — personal skills belong in the `qunm00/pi-skills` git package, project-specific skills in `<workspace>/.pi/skills`

## Learned Workspace Facts

- Project `sbx-pi-kit` is a Docker Sandbox kit for running the `pi` coding agent; image is `nmiquan/sbx-pi-template:latest` (base `docker/sandbox-templates:shell`, runs as non-root `agent`)
- Workspace is bind-mounted at its absolute host path inside the sandbox
- Launcher script `pi-sbx` creates per-directory sandboxes (`pi-<dirname>`); usage is `pi-sbx [kit] [dir]`, where the kit is the first positional argument (`sbx create <kit> --name foo pi <dir>`) and `--kit` is a hard error
- Key sbx commands: `sbx ls`, `sbx create`, `sbx run`, `sbx exec`, `sbx rm`; kit args via `--kit-arg` / `--kit-args-file`
- Sandboxes are ephemeral per-directory: only the image and the bind-mounted workspace persist, and in-sandbox installs like `pi update --extensions` are lost on recreate
- Durable toolchain belongs in the image; project-specific tools belong in a workspace-mounted bootstrap script
- `https://github.com/qunm00/pi-skills` is a git package (entry `git:github.com/qunm00/pi-skills`) with skills under `skills/`, declared in `package.json` via `pi.skills`; it is cloned per sandbox to `~/.pi/agent/git/github.com/qunm00/pi-skills/`
- Each sandbox holds an independent clone frozen at the commit current at create time, so sandboxes drift apart; resync one with `sbx exec <name> pi update --extensions` (git refs can also be pinned to a tag)
- Never author skills directly in `~/.pi/agent/skills/` (vanishes on recreate) or inside the package clone (unpushed edits discarded by `pi update`); promote skills from the host checkout of `pi-skills` instead
- Packages installed via `~/.pi/agent/settings.json`: `git:github.com/qunm00/pi-continual-learning`, `npm:@tintinweb/pi-subagents`, `npm:@upstash/context7-pi`, and `git:github.com/qunm00/pi-skills`
- pi-subagents discovers agent types only from `.pi/agents/`, `.agents/agents/`, and `~/.pi/agent/agents/`
- `.pi/` holds `state/`, `sessions/`, and `patches/`; `.pi/skills/` is currently empty of project skills
- Sources: kit `git+https://github.com/qunm00/sbx-pi-kit.git`, template `https://github.com/qunm00/sbx-pi-template`
