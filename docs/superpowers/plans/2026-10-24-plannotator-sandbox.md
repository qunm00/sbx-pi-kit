# Plannotator in the sbx sandbox — implementation plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Make Plannotator's review UI reachable from the host browser when `pi` runs in the sbx microVM, and keep the extension installed across `--new`.

**Architecture:** `pi-sbx` publishes sandbox port 19432 on the host and starts pi with `PLANNOTATOR_REMOTE=1`, `PLANNOTATOR_PORT=19432`, `PLANNOTATOR_BROWSER=none`, so Plannotator binds `0.0.0.0:19432` and prints `http://localhost:19432`. The template image gains one `pi install` line so a recreated sandbox still has the extension.

**Tech Stack:** bash 3.2 (launcher), Dockerfile (template image), npm package `@plannotator/pi-extension`.

**Spec:** `docs/superpowers/specs/2026-10-24-plannotator-sandbox-design.md`

## Global Constraints

- `pi-sbx` must stay bash 3.2 compatible: expand arrays as `${arr[@]+"${arr[@]}"}`.
- Review port is `19432` on both sides; remote mode is unconditional.
- `PLANNOTATOR_BROWSER=none`; no browser exists in the sandbox.
- The template repo (`sbx-pi-template`) is **not** mounted in this sandbox; it is patched with a `git apply`-able diff.
- No behavior change for sandboxes that never use Plannotator: publish failure is a warning, not an error.

## Review Focus

- Second sandbox with the same name family tries to publish 19432 while another holds it: host bind fails; launcher must warn and continue.
- Host already has something on 19432: same path.
- `sbx ports` output format differs or command is missing on older sbx: no crash, warning only.
- Re-running `pi-sbx` on an existing sandbox must not re-publish or error.
- `PI_SBX_DRY_RUN=1` must stay side-effect free and still show the new commands.

---

### Task 1: Publish the review port and set Plannotator env in `pi-sbx`

**Files:**
- Modify: `pi-sbx` (after the `EXISTS` probe, and in `exec_pi`)

**Interfaces:**
- Produces: env vars `PLANNOTATOR_REMOTE`, `PLANNOTATOR_PORT`, `PLANNOTATOR_BROWSER` in the pi process; host mapping `127.0.0.1:19432 -> 19432`.

- [ ] **Step 1: Add the constants and publish helper**

Near the other settings, add:

```bash
PLANNOTATOR_PORT=19432
```

and, after `EXISTS` is known:

```bash
# Plannotator's review UI runs inside the sandbox; the browser runs on the
# host, so the fixed review port must be published. Idempotent: re-publishing
# an existing mapping is skipped, and a failure only warns.
publish_review_port() {
  local name="$1" mapping
  mapping="${PLANNOTATOR_PORT}:${PLANNOTATOR_PORT}"
  if [ "$DRY_RUN" = "1" ]; then
    log "${C_DIM}[dry-run]${C_RST} sbx ports $(printf '%q' "$name") --publish ${mapping}"
    return 0
  fi
  if sbx ports "$name" 2>/dev/null | grep -q -- "->${PLANNOTATOR_PORT}/"; then
    return 0
  fi
  if ! sbx ports "$name" --publish "$mapping" >/dev/null 2>&1; then
    log "${C_YELLOW}warning:${C_RST} could not publish port ${PLANNOTATOR_PORT}; Plannotator reviews will not open in the host browser"
    log "         check 'sbx ports ${name}' and any process already using ${PLANNOTATOR_PORT}"
  fi
}
```

- [ ] **Step 2: Call it once, after the sandbox exists**

Right after `if ! $EXISTS; then ... fi` (both create and reattach paths converge there), before `sleep 2`:

```bash
publish_review_port "$SANDBOX_NAME"
```

- [ ] **Step 3: Inject the env vars in `exec_pi`**

```bash
  sbx exec -it "$SANDBOX_NAME" \
    env PI_CODING_AGENT_SESSION_DIR="$SESSION_DIR" \
        PLANNOTATOR_REMOTE=1 \
        PLANNOTATOR_PORT="$PLANNOTATOR_PORT" \
        PLANNOTATOR_BROWSER=none \
    sh -c 'cd "$1" 2>/dev/null || cd "$HOME" || cd /; shift; exec "$@"' \
    sh "$WORKDIR" pi --model opencode/space-bunny-free
```

Mirror the same three vars in the `DRY_RUN` echo below it, then add a line to the banner:

```bash
log "${C_CYAN}│${C_RST} review    : http://localhost:${PLANNOTATOR_PORT} (Plannotator)"
```

- [ ] **Step 4: Verify with a dry run from the kit directory**

Run: `PI_SBX_DRY_RUN=1 ./pi-sbx . 2>&1 | tail -20`
Expected: a `[dry-run] sbx ports 'pi-sbx' --publish 19432:19432` line, and the `sbx exec ... PLANNOTATOR_REMOTE=1 PLANNOTATOR_PORT=19432 PLANNOTATOR_BROWSER=none ...` line.

Run: `PI_SBX_DRY_RUN=1 ./pi-sbx . 2>&1 | grep PLANNOTATOR`
Expected: one line, the `sbx exec ... env ... PLANNOTATOR_REMOTE=1 PLANNOTATOR_PORT=19432 PLANNOTATOR_BROWSER=none ...` echo.

Run: `bash -n pi-sbx && command -v shellcheck >/dev/null && shellcheck pi-sbx`
Expected: no syntax error; shellcheck clean or only pre-existing findings.

- [ ] **Step 5: Commit**

```bash
git add pi-sbx
git commit -m "feat(pi-sbx): publish Plannotator review port and enable remote mode"
```

---

### Task 2: Template image patch (external repo)

**Files:**
- Create: `docs/patches/sbx-pi-template-plannotator.diff`
- Modify: `README.md` (Template section, Task 3 covers the wording)

**Interfaces:**
- Produces: a diff that applies cleanly to `qunm00/sbx-pi-template` at the commit currently on `main`.

- [ ] **Step 1: Clone the template repo to a scratch dir and confirm the anchor lines**

```bash
git clone --depth 1 https://github.com/qunm00/sbx-pi-template /tmp/sbx-pi-template
grep -n "pi install" /tmp/sbx-pi-template/Dockerfile
```

Expected: the `pi install` block, ending with `pi install https://github.com/umputun/revdiff`.

- [ ] **Step 2: Make the edit in the scratch clone**

In `Dockerfile`, after the `SimpleEnglish` line add:

```dockerfile
    && pi install npm:@plannotator/pi-extension \
```

and after `ENV PATH="/home/agent/.npm-global/bin:${PATH}"` add:

```dockerfile
# Plannotator's review UI is opened from the host browser through a published
# port, so it must bind all interfaces and must not try to launch a browser.
ENV PLANNOTATOR_REMOTE=1 \
    PLANNOTATOR_PORT=19432 \
    PLANNOTATOR_BROWSER=none
```

- [ ] **Step 3: Emit the patch into the kit repo**

```bash
cd /tmp/sbx-pi-template && git diff > /Users/quannm/Projects/sbx-pi-kit/docs/patches/sbx-pi-template-plannotator.diff
```

- [ ] **Step 4: Verify the patch applies**

```bash
cd /tmp && rm -rf verify-tpl && git clone --depth 1 https://github.com/qunm00/sbx-pi-template verify-tpl
cd verify-tpl && git apply --check /Users/quannm/Projects/sbx-pi-kit/docs/patches/sbx-pi-template-plannotator.diff
```

Expected: exit 0, no output.

- [ ] **Step 5: Commit**

```bash
cd /Users/quannm/Projects/sbx-pi-kit
git add docs/patches/sbx-pi-template-plannotator.diff
git commit -m "docs(plannotator): template image patch for the pi extension"
```

---

### Task 3: README section

**Files:**
- Modify: `README.md`

- [ ] **Step 1: Add a Plannotator section after "Template"**

Cover, in the repo's voice:

- What it is: plan review, annotate, code review in the host browser.
- Why nothing extra is needed: `pi-sbx` publishes 19432 and sets remote mode.
- Usage: `Ctrl+Alt+P` or `/plannotator-plan-mode`, then open
  `http://localhost:19432`; `/plannotator-annotate <file>`,
  `/plannotator-last`, `/plannotator-review`.
- If the URL does not open: `sbx ports pi-sbx` must show
  `127.0.0.1:19432->19432/tcp4`, and only one sandbox can hold that port.
- The extension lives in the image; apply
  `docs/patches/sbx-pi-template-plannotator.diff` in the template checkout,
  then `./rebuild.sh`.

- [ ] **Step 2: Verify**

Run: `grep -n "Plannotator" README.md`
Expected: the new section heading and the port line.

- [ ] **Step 3: Commit**

```bash
git add README.md
git commit -m "docs: how to use Plannotator from the sandbox"
```
