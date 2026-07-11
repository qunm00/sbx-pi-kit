# sbx-pi-kit

Docker Sandbox kit for running the `pi` coding agent with:
- OpenRouter free-tier models
- GitHub Copilot (existing subscription)

## Usage

### Quick start with shell alias

Add this to your `~/.zshrc`:

```bash
alias sbx-pi='sbx run --kit git+https://github.com/qunm00/sbx-pi-kit.git pi'
```

Then reload:
```bash
source ~/.zshrc
```

Set up your secret once:
```bash
sbx secret set OPENROUTER_API_KEY sk-or-...
```

Now you can run:
```bash
sbx-pi ~/my-project
```

### Manual command (without alias)

```bash
sbx secret set OPENROUTER_API_KEY sk-or-...
sbx run --name my-project-pi \
  --kit git+https://github.com/qunm00/sbx-pi-kit.git \
  pi \
  ~/my-project
```

Reattach later without `--kit`:

```bash
sbx run my-project-pi
```

## Template

Built from [sbx-pi-template](https://github.com/qunm00/sbx-pi-template).