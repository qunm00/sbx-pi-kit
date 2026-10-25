# Visual companion in the sbx sandbox — design

Date: 2026-10-25
Status: proposed

## Intent

Make the superpowers visual companion (the brainstorm server) reachable from
the host browser when pi runs inside the sbx sandbox, the same way Plannotator
is.

## Why it does not work today

The companion starts on demand when the agent runs start-server.sh. It binds
127.0.0.1 on a random high port. A published port cannot reach a loopback bind,
and a random port matches no published mapping. pi-sbx publishes only 19432 for
Plannotator.

## Goals

- The companion opens at http://localhost:19433 on the host.
- The setup survives sandbox recreation (--new) and applies to every project
  the kit creates.

## Non-goals

- No per-project .pi/skills file.
- No template image change.
- No edit inside the superpowers package (pi update overwrites it).

## Decisions

| Decision | Choice | Reason |
| --- | --- | --- |
| Port | 19433 on both sides | One stable URL, like Plannotator's 19432 |
| Publish | pi-sbx, sbx ports --publish 19433:19433, idempotent | Same pattern as Plannotator |
| Pin the port | spec.yaml environment.variables, BRAINSTORM_PORT=19433 | Inherited by the server; start-server.sh does not override it |
| Bind every interface | spec.yaml memory, one line: start with --host 0.0.0.0 --url-host localhost | start-server.sh hard-codes 127.0.0.1 and ignores an inherited bind host, so a flag is required; kit memory reaches every agent |

## Constraints and risks

- One sandbox can hold host port 19433 at a time. A second named sandbox fails
  to publish; pi-sbx warns, like Plannotator.
- A fixed port allows one companion session at a time.
- The agent must pass the --host flag; the kit memory is the only reminder.
