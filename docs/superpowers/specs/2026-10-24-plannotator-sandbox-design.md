# Plannotator in the sbx sandbox — design

Date: 2026-10-24
Status: proposed

## Intent

Make Plannotator's browser review UI usable when `pi` runs inside the sbx
microVM, so plans, `annotate` targets, and code reviews open in the host
browser and the decision comes back to the agent.

## Why it does not work today

`@plannotator/pi-extension@0.28.30` is installed and loads fine, but the
extension runs inside the sandbox while the browser runs on the host:

- Without `PLANNOTATOR_REMOTE`, the review server binds `127.0.0.1` on a
  random port. The host cannot reach either.
- `pi-sbx` publishes no ports, so even a fixed port is unreachable.
- There is no browser inside the sandbox; Plannotator's auto-open is wasted.

Plannotator's own Docker guidance is `PLANNOTATOR_REMOTE=1`,
`PLANNOTATOR_PORT=<fixed>`, publish that port, and skip browser opening.

## Goals

- Plan review, `/plannotator-annotate`, `/plannotator-last`, and
  `/plannotator-review` open at `http://localhost:19432` on the host.
- The setup survives sandbox recreation (`--new`) and a fresh machine
  building the template image.

## Non-goals

- Plannotator Inbox (`plannotator inbox`): needs the standalone CLI binary
  and host-side state, not covered here.
- Any per-phase model/tool config in `~/.pi/agent/plannotator.json`.
- Publishing ports other than the review port.

## Decisions

| Decision | Choice | Reason |
| --- | --- | --- |
| Mode | Always remote | Browser is always on the host; set `PLANNOTATOR_REMOTE=1` unconditionally |
| Port | 19432 on both sides | Plannotator's remote default; one bookmark; fixed was requested |
| Browser | `PLANNOTATOR_BROWSER=none` | No browser in the microVM; keeps output to a printed URL |
| Where ports are published | `pi-sbx` via `sbx ports <name> --publish 19432:19432` | Host-side action; kit stays the single entry point |
| Where env lives | `pi-sbx` `sbx exec` env line, plus image `ENV` | Launcher always wins; image `ENV` still helps bare `sbx exec` |
| Package durableness | `pi install npm:@plannotator/pi-extension` in the template image | `~/.pi/agent` is per-sandbox; `--new` wipes it |

## How it works

1. `pi-sbx` publishes `19432:19432` for the sandbox if that mapping is not
   already present (checked from `sbx ports` output, so re-runs are cheap).
2. `pi-sbx` starts pi with `PLANNOTATOR_REMOTE=1`,
   `PLANNOTATOR_PORT=19432`, `PLANNOTATOR_BROWSER=none`.
3. Plan mode (`Ctrl+Alt+P` or `/plannotator-plan-mode`) binds `0.0.0.0:19432`
   and prints `http://localhost:19432`. The host browser opens it through
   the published mapping; approvals and annotations POST back to the same
   server and reach the agent as its next message.
4. The Host-header guard needs no extra allowlist: forwarded requests arrive
   as `localhost`.

## Constraints and risks

- Port publishing binds `127.0.0.1` on the host. One sandbox at a time can
  hold 19432; a second named sandbox fails to publish. The launcher warns.
- Remote mode disables: Ask AI "Ask this session", the annotate agent
  terminal, Open in, Update now, live app annotation. Plan review, annotate,
  and code review keep working.
- `sbx ports --publish` requires a `sbx` new enough to have the command.

## Left to the builder

- Exact shell in `pi-sbx` (bash 3.2 safe, dry-run aware).
- The Dockerfile line plus a `git apply`-able patch for the template repo,
  which is not mounted inside the sandbox.
- README section with the commands.
