# Visual companion sandbox port — implementation plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Publish a fixed companion port and pin the server so the host browser reaches it at http://localhost:19433.

**Architecture:** pi-sbx publishes sandbox port 19433 on the host, idempotently, next to Plannotator's 19432. spec.yaml sets BRAINSTORM_PORT=19433 so the server inherits the fixed port, and adds one memory line telling the agent to start the companion with --host 0.0.0.0 --url-host localhost.

**Tech Stack:** bash 3.2 (launcher), YAML (kit spec).

**Spec:** `docs/superpowers/specs/2026-10-25-visual-companion-sandbox-design.md`

## Global Constraints

- pi-sbx must stay bash 3.2 compatible: expand arrays as `${arr[@]+"${arr[@]}"}`.
- Companion port is 19433 on both sides.
- Kit environment and memory are stored at sandbox creation; apply with `pi-sbx --new`.
- Publish failure is a warning, not an error.

## Review Focus

- A second named sandbox publishes 19433 while another holds it: host bind fails; pi-sbx warns and continues.
- sbx ports output differs or the command is missing on older sbx: no crash, warning only.
- Re-running pi-sbx on an existing sandbox must not re-publish or error.
- PI_SBX_DRY_RUN=1 must stay side-effect free and still show the new commands.
- An existing sandbox keeps old env and memory until --new.

---

### Task 1: Publish the companion port in pi-sbx

**Files:**
- Modify: `pi-sbx`

**Interfaces:**
- Produces: host mapping `127.0.0.1:19433 -> 19433`; a `companion` banner line.

- [ ] **Step 1: Generalize `publish_review_port` into `publish_port`**

Add `COMPANION_PORT=19433` next to `PLANNOTATOR_PORT`, and replace the
function with one that takes a name, a port, and a label. The label goes in the
warning text.

- [ ] **Step 2: Call it for both ports**

Replace the single `publish_review_port "$SANDBOX_NAME"` call with two calls:
one for `$PLANNOTATOR_PORT` labelled "Plannotator reviews", one for
`$COMPANION_PORT` labelled "the visual companion".

- [ ] **Step 3: Add the banner line**

After the Plannotator review line, add:

```bash
log "${C_CYAN}│${C_RST} companion : http://localhost:${COMPANION_PORT} (visual companion)"
```

- [ ] **Step 4: Verify the launcher**

Run: `bash -n pi-sbx && (command -v shellcheck >/dev/null && shellcheck pi-sbx || true)`
Expected: no syntax error; shellcheck clean or only pre-existing findings.

Run: `PI_SBX_DRY_RUN=1 ./pi-sbx . 2>&1 | grep -E 'publish 1943|companion'`
Expected: two `sbx ports ... --publish` dry-run lines (19432 and 19433) and the
`companion` banner line.

- [ ] **Step 5: Commit**

```bash
git add pi-sbx
git commit -m "feat(pi-sbx): publish the visual companion port"
```

---

### Task 2: Pin the port and the instruction in spec.yaml

**Files:**
- Modify: `spec.yaml`

**Interfaces:**
- Produces: env var `BRAINSTORM_PORT=19433` in the sandbox; a memory line that
  tells the agent to pass `--host 0.0.0.0 --url-host localhost`.

- [ ] **Step 1: Add the env var**

Under `environment.variables`, after `PLANNOTATOR_BROWSER`, add:

```yaml
    BRAINSTORM_PORT: "19433"
```

- [ ] **Step 2: Add the memory line**

Under `memory`, after the providers line, add:

```text
  The visual companion (the superpowers brainstorm server) must start with
  --host 0.0.0.0 --url-host localhost so the published port 19433 reaches
  the host browser.
```

- [ ] **Step 3: Verify**

Run: `grep -n "BRAINSTORM_PORT\|--host 0.0.0.0" spec.yaml`
Expected: one `BRAINSTORM_PORT` line and one `--host 0.0.0.0` line.

- [ ] **Step 4: Commit**

```bash
git add spec.yaml
git commit -m "feat(spec): pin and expose the visual companion port"
```

---

### Task 3: README section

**Files:**
- Modify: `README.md`

- [ ] **Step 1: Add a Visual companion section after the Plannotator section**

Cover, in the repo's voice: what it is, that pi-sbx publishes 19433 and pins
BRAINSTORM_PORT, the URL http://localhost:19433, the two flags the server needs,
and that an existing sandbox needs `pi-sbx --new` to pick up the new env and
memory.

- [ ] **Step 2: Verify**

Run: `grep -n "19433\|Visual companion" README.md`
Expected: the new section heading and the port line.

- [ ] **Step 3: Commit**

```bash
git add README.md
git commit -m "docs: how to use the visual companion from the sandbox"
```
