# sbx-pi-kit

Docker Sandbox kit for running the `pi` coding agent with:
- OpenRouter free-tier models
- OpenCode

## Prerequisites

```bash
brew install docker/tap/sbx
sbx login
```
### Set secrets on host

```bash
sbx secret set-custom -g \
  --host api.openrouter.ai \
  --env OPENROUTER_API_KEY

sbx secret set-custom -g \
  --host api.opencode.ai \
  --env OPENCODE_API_KEY

sbx secret set-custom -g \
    --host context7.com \
    --env CONTEXT7_API_KEY
```

Retrieve the placeholder secret key from the command then place it in `spec.yaml`

## Usage

### Launcher script (recommended)

```bash
# Symlink the launcher onto your PATH
ln -sf "$PWD/pi-sbx" ~/.local/bin/pi-sbx

# Run in current directory
pi-sbx

# Run with specific workspace
pi-sbx ~/my-project

# Force a fresh sandbox
pi-sbx --new ~/my-project

# Use a local kit checkout (kit is the first positional argument)
pi-sbx ../sbx-pi-kit ~/my-project
```

> The kit is the first positional argument (`pi-sbx <kit> [dir]`). The old
> `--kit` flag was removed.

The launcher uses one shared sandbox named `pi-sbx` for every project. Running
`pi-sbx` again from anywhere re-uses it. Use `--new` to start fresh.

### One sandbox for many projects

`PI_SBX_MOUNTS` lists the directories to mount. When it is set, those
directories are the only mounts and the current directory is ignored. When it
is unset, the launcher mounts the workspace directory instead.

```bash
# Mount ~/work and ~/shared-libs (read-only) and nothing else
PI_SBX_MOUNTS="~/work,~/shared-libs:ro" pi-sbx

# Same, for every shell: put it in ~/.zshrc
export PI_SBX_MOUNTS="~/work,~/shared-libs:ro"
```

Sessions are stored outside any project so all projects share them:

```bash
export PI_SBX_SESSIONS=~/.pi-sbx   # default
```

The launcher mounts this directory read-write and points pi at
`<dir>/sessions`. It also records the mount list at `<dir>/sbx-mounts`, so a
changed mount list warns and suggests `--new` (sbx fixes mounts at create
time).

> `--new` destroys the shared sandbox for every project. Re-run it from the
> directory whose mount list you want.

### Manual sbx commands

```bash
# First run (creates sandbox and set custom secret)
# The kit is the first positional argument; --kit is deprecated
sbx create "git+https://github.com/qunm00/sbx-pi-kit.git" --name pi-sbx pi ~/my-project ~/.pi-sbx
sbx create . --name pi-sbx pi ~/my-project ~/.pi-sbx

# Extra mounts: paths after the workspace, mounted at the same path (:ro = read-only)
sbx create . --name pi-sbx pi ~/my-project ~/.pi-sbx ~/shared-libs:ro

# Re-attach later
sbx run "git+https://github.com/qunm00/sbx-pi-kit.git" --name pi-sbx
sbx run . --name pi-sbx

# One-shot prompt
sbx exec pi-sbx pi -p "list all .ts files"

# Destroy
sbx rm -f pi-sbx
```

### One-shot prompts via alias

```bash
alias sbx-pi='sbx exec pi-sbx pi'
sbx-pi -p "fix the bug in src/main.ts"
```

## How it works

The [pi-sbx](pi-sbx) launcher handles a two-phase lifecycle:

1. `sbx create` — starts the sandbox microVM with the workspace, the sessions
   directory, and any `PI_SBX_MOUNTS` directories
2. `sbx exec` — attaches your terminal (provides TTY for pi's interactive TUI)

## Template

Built from [sbx-pi-template](https://github.com/qunm00/sbx-pi-template).
