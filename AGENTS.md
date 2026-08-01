## Learned User Preferences

- Keep sandbox-isolated user namespace (`~/.pi/agent/`) separate from synced project namespace (`<workspace>/.pi/`)
- Sync only sessions into the project folder, not full config with auth tokens
- Manage credentials through sbx secrets (env vars) rather than pi's auth.json
- Use `PI_CODING_AGENT_SESSION_DIR` to redirect session storage to the mounted workspace
- Use `.pi/settings.json` for project settings that persist across sandbox instances

## Learned Workspace Facts

- Project `sbx-pi-kit` is a Docker Sandbox kit for running the `pi` coding agent
- Sandbox image: `nmiquan/sbx-pi-template:latest`
- Workspace is bind-mounted at its absolute host path inside the sandbox
- Available providers: OpenRouter free-tier and OpenCode
- Package source: `git+https://github.com/qunm00/sbx-pi-kit.git`
- Template source: `https://github.com/qunm00/sbx-pi-template`
- Launcher script `pi-sbx` creates per-directory sandboxes (`pi-<dirname>`)
- Key sbx commands: `sbx ls`, `sbx create`, `sbx run`, `sbx exec`, `sbx rm`
- `.pi/` directory contains `skills/` and `state/` subdirectories
- Model in use: `opencode/deepseek-v4-flash-free` with high thinking level
