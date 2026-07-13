# sbx-pi-kit

Docker Sandbox kit for running the `pi` coding agent with:
- OpenRouter free-tier models
- GitHub Copilot (existing subscription)

## Prerequisites

```bash
brew install docker/tap/sbx
sbx login
sbx secret set OPENROUTER_API_KEY sk-or-...
```

## Usage

### Login OpenRouter

```bash
sbx secret set-custom -g \
  --host api.openrouter.ai \
  --env OPENROUTER_API_KEY
```

Retrieve the placeholder secret key from the command then place it in `spec.yaml`

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
```

The launcher creates a per-directory sandbox (`pi-<dirname>`). Running from the
same directory reconnects to the same sandbox. Use `--new` to start fresh.

### Manual sbx commands

```bash
# First run (creates sandbox and set custom secret)
sbx create --kit "git+https://github.com/qunm00/sbx-pi-kit.git" --name my-project-pi pi ~/my-project
sbx create --kit . --name my-project-pi pi ~/my-project

# Re-attach later
sbx run --kit "git+https://github.com/qunm00/sbx-pi-kit.git" --name my-project-pi
sbx run --kit . --name my-project-pi

# One-shot prompt
sbx exec my-project-pi pi -p "list all .ts files"

# Destroy
sbx rm -f my-project-pi
```

### One-shot prompts via alias

```bash
alias sbx-pi='sbx exec $(sbx ls 2>/dev/null | awk "NR>1 && /^pi-/ {print \$1; exit}") pi'
sbx-pi -p "fix the bug in src/main.ts"
```

## How it works

The [pi-sbx](pi-sbx) launcher handles a two-phase lifecycle:

1. `sbx create` — starts the sandbox microVM with your workspace mounted
2. `sbx run` — attaches your terminal (provides TTY for pi's interactive TUI)

## Template

Built from [sbx-pi-template](https://github.com/qunm00/sbx-pi-template).
