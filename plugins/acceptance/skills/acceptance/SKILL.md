---
name: acceptance
description: Gather acceptance criteria interactively, agree evaluation methods with the user, store them in .claude/acceptance/, then verify each criterion and update the file with evidence. Supports workflows and resuming interrupted runs.
argument-hint: [criterion] | resume | workflow <name> | workflow save <name> | workflow edit <name> | workflow delete <name>
disable-model-invocation: true
---

You are running an acceptance verification session. Your job is to evaluate **outcomes** — observable behaviour of the running system — not code quality, structure, or style. You may invoke linters or quality tools only when a criterion explicitly requires it (e.g. "the build must pass CI"). Never make judgements about code quality as an end in itself.

---

## Entry point — read this first, follow only one path

**IMPORTANT: these are completely separate flows. Execute only the one that matches. Never mix flows.**

### If `$ARGUMENTS` is empty:
→ Stop here. Go to **Menu flow**. Do not gather criteria. Do not scan for runs.

### If `$ARGUMENTS` starts with `workflow save`:
→ Stop here. Go to **Workflow: save flow**. Do not gather new criteria. Do not create a run file.

### If `$ARGUMENTS` starts with `workflow edit`:
→ Stop here. Go to **Workflow: edit flow**. Do not gather new criteria. Do not create a run file.

### If `$ARGUMENTS` starts with `workflow delete`:
→ Stop here. Go to **Workflow: delete flow**. Do not gather criteria. Do not create any file.

### If `$ARGUMENTS` starts with `workflow` (but not save/edit/delete):
→ Stop here. Go to **Workflow: run flow**. Do not gather criteria interactively. Do not scan for existing runs.

### If `$ARGUMENTS` is exactly `resume` or starts with `resume`:
→ Stop here. Go to **Resume flow**. Do not gather criteria. Do not create a new file.

### If `$ARGUMENTS` is anything else (non-empty, not matching above):
→ Stop here. Go to **New session flow**. Do not scan `.claude/acceptance/`. Do not list existing runs.

---

## Menu flow — only reached via `/acceptance` with no arguments

Display this menu exactly:

```
Acceptance Testing

  1. New Task
  2. Workflow - Run
  3. Workflow - Create
  4. Workflow - Edit
  5. Workflow - Delete
  6. Resume

Pick an option:
```

Wait for the user to enter a number. Then:

- **1** → Go to **New session flow** (ask for the first criterion).
- **2** → Ask: "Which workflow would you like to run?" List available workflows from `.claude/acceptance/workflows/`. Then proceed as **Workflow: run flow** for the chosen name.
- **3** → Go to **Workflow: create flow**.
- **4** → Ask: "Which workflow would you like to edit?" List available workflows from `.claude/acceptance/workflows/`. Then proceed as **Workflow: edit flow** for the chosen name.
- **5** → Ask: "Which workflow would you like to delete?" List available workflows from `.claude/acceptance/workflows/`. Then proceed as **Workflow: delete flow** for the chosen name.
- **6** → Go to **Resume flow**.
- Anything else → say "Invalid option. Please pick 1, 2, 3, 4, 5, or 6." and re-display the menu.

---

## Workflow: create flow — only reached via menu option 3

This flow creates a new workflow file from scratch. **Do not create a run file. Do not run verification.**

1. Ask: "What would you like to name this workflow?"

2. Gather criteria one at a time using the same process as Phase 1 of the new session flow (rewrite as observable outcome, propose method, confirm). After each criterion is confirmed, ask: **"Should this criterion run Before or After the user's own criteria?"** Record the answer (Before/After) alongside each criterion.

   Continue until the user says 'done'.

3. Ensure `.claude/acceptance/workflows/` exists (create if not).

4. Write the workflow file at `.claude/acceptance/workflows/<name>.md`, placing criteria in the correct section based on their Before/After assignment:

```markdown
---
name: [name]
created: [ISO timestamp]
---

# Workflow: [name]

[Optional one-line description of when to use this workflow]

## Before

### 1. [Short title]

**Criterion:** [Observable outcome statement]
**Method:** [Agreed evaluation steps and tools]

---

## After

### 1. [Short title]

**Criterion:** [Observable outcome statement]
**Method:** [Agreed evaluation steps and tools]
```

5. Confirm: "Workflow '[name]' saved to `.claude/acceptance/workflows/<name>.md`. Run it with `/acceptance workflow <name>`." Stop — do not proceed to verification.

---

## Workflow: run flow — only reached via `/acceptance workflow <name>`

Extract the workflow name from `$ARGUMENTS` (everything after `workflow `).

1. Look for the workflow file at `.claude/acceptance/workflows/<name>.md`. If it does not exist, tell the user: "No workflow named '<name>' found at `.claude/acceptance/workflows/<name>.md`." Offer to list available workflows or start a new session.

2. Load the workflow file. It contains three sections:
   - `## Before` — workflow-defined criteria that run first
   - `## After` — workflow-defined criteria that run last
   - Optionally a description

3. Present the workflow's before and after criteria to the user so they can see what's already defined.

4. Gather any additional user criteria using the same Phase 1 process as the new session flow — rewrite each as an observable outcome, propose a method, confirm with the user. These will run between before and after. Skip this step if the user says the workflow criteria are sufficient on their own.

5. Proceed to **Phase 2** to create the run file. The criteria order in the file must be: workflow before criteria → user criteria → workflow after criteria. All criteria use the same format regardless of source.

6. Continue through Phases 3, 4, and 5 as normal.

---

## Workflow: save flow — only reached via `/acceptance workflow save <name>`

Extract the workflow name from `$ARGUMENTS` (everything after `workflow save `).

1. Check if there is an active acceptance run in the current session with confirmed criteria. If not, ask the user to first gather criteria in a new session before saving.

2. Ask the user which criteria to include in the workflow's `before` section and which in the `after` section. Any unassigned criteria are not saved to the workflow.

3. Ensure `.claude/acceptance/workflows/` exists (create if not).

4. Write the workflow file at `.claude/acceptance/workflows/<name>.md`:

```markdown
---
name: [name]
created: [ISO timestamp]
---

# Workflow: [name]

[Optional one-line description of when to use this workflow]

## Before

### 1. [Short title]

**Criterion:** [Observable outcome statement]
**Method:** [Agreed evaluation steps and tools]

---

## After

### 1. [Short title]

**Criterion:** [Observable outcome statement]
**Method:** [Agreed evaluation steps and tools]
```

5. Confirm to the user: "Workflow '[name]' saved to `.claude/acceptance/workflows/<name>.md`. Run it with `/acceptance workflow <name>`."

---

## Workflow: edit flow — only reached via `/acceptance workflow edit <name>`

Extract the workflow name from `$ARGUMENTS` (everything after `workflow edit `).

1. Check the workflow file exists at `.claude/acceptance/workflows/<name>.md`. If not, tell the user it wasn't found and list available workflows.

2. Load and display the current before and after criteria.

3. Re-enter the criteria-gathering loop pre-populated with the existing criteria. For each existing criterion, show it and ask: "Keep as-is, modify, or remove?" Allow the user to add new criteria and assign them to before or after.

4. Once the user says done, overwrite the workflow file with the updated content (same format as save flow). Confirm the file has been updated.

---

## Workflow: delete flow — only reached via `/acceptance workflow delete <name>`

Extract the workflow name from `$ARGUMENTS` (everything after `workflow delete `).

1. Check the workflow file exists at `.claude/acceptance/workflows/<name>.md`. If not, tell the user it wasn't found.

2. Ask for confirmation: "Delete workflow '<name>'? This cannot be undone. (yes/no)"

3. On confirmation, delete the file. Confirm: "Workflow '<name>' deleted."

4. On anything other than yes, cancel and do nothing.

---

## Resume flow — only reached via `/acceptance resume`

1. Ensure `.claude/acceptance/` exists. If the directory is missing or empty, tell the user there are no saved runs and offer to start a new one.

2. Scan all `.md` files in `.claude/acceptance/` (not in the `workflows/` subdirectory). For each file, read the frontmatter `status` field and count criteria by their Status values.

3. List only files where `status: IN_PROGRESS`. Format:

```
Unfinished acceptance runs:

  1. 2026-03-18 14:32 — form-validation.md
     3 criteria · 1 passed · 2 pending

  2. 2026-03-17 09:15 — checkout-flow.md
     5 criteria · 3 passed · 2 pending

Pick a number to resume, or 'new' to start a fresh session.
```

4. Wait for the user to pick. Load the chosen file, identify all criteria with `Status: PENDING`, and continue from **Phase 4 — Verification** for those criteria only. Already-passed criteria are not re-run unless the user asks.

---

## New session flow — only reached via `/acceptance` or `/acceptance [criterion]`

### Phase 1 — Gather criteria

Work through criteria one at a time. If `$ARGUMENTS` contains a criterion, use it as the first one — otherwise ask for it.

For **each** criterion:

1. **Receive** the criterion from the user
2. **Rewrite** it as a specific, observable outcome:
   - Bad: "the form should work"
   - Good: "submitting an empty form shows inline validation errors without calling the API"
   - Note any assumption inline if the original was ambiguous
3. **Propose an evaluation method** — the exact steps and tools you will use. Be concrete:
   - Bad: "I'll test the form"
   - Good: "I'll open the form in Chrome DevTools, submit with all fields empty, screenshot the result to confirm errors appear on each required field, and verify in the Network tab that no API call was made"
4. **Confirm with the user** — "Does this criterion and method look right, or do you want to adjust?" Wait for their response.
5. Once confirmed, ask: "What's the next criterion? (or say 'done' to begin verification)"

Repeat until the user says done.

---

### Phase 2 — Create the acceptance file

Generate a unique filename using the current timestamp and a short slug from the first criterion:

```bash
date +%Y%m%d-%H%M%S
```

Slug rules: lowercase, hyphens only, max 30 chars, derived from the first criterion (e.g. "form validates on submit" → `form-validates-on-submit`).

Final filename example: `20260318-143200-form-validates-on-submit.md`

Ensure `.claude/acceptance/` exists (create it if not). Write the file there.

File format:

```markdown
---
started: [ISO timestamp]
status: IN_PROGRESS
---

# Acceptance: [slug]

_Each criterion has an agreed evaluation method. Status is updated in real time during verification._

---

## 1. [Short title]

**Criterion:** [Observable outcome statement]
**Method:** [Agreed evaluation steps and tools]
**Status:** PENDING
**Evidence:** —

---

## 2. [Short title]

**Criterion:** [...]
**Method:** [...]
**Status:** PENDING
**Evidence:** —
```

Confirm the file path to the user. Then ask: "Ready to run verification?"

---

### Phase 3 — Tool verification

Before running any verifications, actively check that every tool required by the agreed methods is available. Do not assume — verify.

#### Step 1: Identify required tool categories

Based on the agreed methods, determine which categories apply:

| Observation type | Primary tool | Check command |
|---|---|---|
| UI / browser behaviour | `chrome-cli` | `which chrome-cli` |
| Browser automation (alt) | Playwright | `npx playwright --version` |
| Browser automation (alt) | Puppeteer | `npx puppeteer --version` |
| HTTP / API responses | `curl` | `which curl` |
| HTTP / API responses (alt) | `httpie` | `which http` |
| Database — Postgres | `psql` | `which psql` |
| Database — MySQL | `mysql` | `which mysql` |
| Database — SQLite | `sqlite3` | `which sqlite3` |
| Build / test suite | project-defined | read `package.json`, `Makefile`, `pyproject.toml` |
| Process / CLI output | `Bash` | always available |
| File system | `Read`, `Glob` | always available |

#### Step 2: Run checks

For each required tool, run its check command. Collect results — available or missing.

Also inspect the project for installed dependencies that could serve as alternatives:
- Check `package.json` → `dependencies`, `devDependencies`, `scripts`
- Check `requirements.txt` or `pyproject.toml` if Python project
- Check `Gemfile` if Ruby, `go.mod` if Go
- Note any testing frameworks, browser automation libs, or HTTP clients already present

#### Step 3: Report findings and resolve gaps

Present a clear status table:

```
Tool check results:
  ✓ curl           — available
  ✓ Bash           — available
  ✗ chrome-cli     — NOT FOUND
    → Playwright detected in package.json devDependencies
    → Alternative: use `npx playwright` for browser verification
    → Or install chrome-cli: brew install chrome-cli
  ✗ psql           — NOT FOUND
    → No database client found in PATH
    → Recommendation: install psql (brew install postgresql) or confirm DB access method
```

For each missing tool, provide:
1. Whether a usable alternative was found in the project
2. An install recommendation if no alternative exists
3. A specific question: "Shall I use [alternative] instead, or do you want to install [tool] first?"

**Do not proceed to Phase 4 until every required tool is either confirmed available or the user has approved an alternative.** If a gap cannot be resolved, tell the user which criteria cannot be verified and ask whether to skip, defer, or halt.

---

### Phase 4 — Verification

Work through each PENDING criterion in order, using the **agreed method**. Do not deviate from the agreed method without telling the user.

For each criterion:
1. **State what you are about to do** (one sentence referencing the agreed method)
2. **Execute the observation** — use the agreed tools, capture real output
3. **Record the evidence** — quote actual output, include screenshot paths, paste response bodies
4. **Mark PASS or FAIL** with a one-line reason grounded in the evidence

If a criterion **fails**:
- Identify the root cause from the observed evidence
- Make targeted fixes and re-verify (up to 3 cycles)
- Every fix must be motivated by observed evidence — no speculative changes
- If related criteria may be affected by a fix, re-verify those too

**After each criterion**, immediately update the file:
- Set `Status:` to `PASS` or `FAIL`
- Fill in `Evidence:` with a concise summary and any screenshot paths
- If all criteria are now resolved, update the frontmatter `status:` to `COMPLETE`

---

### Phase 5 — Final report

Output a summary to the terminal:

```
ACCEPTANCE REPORT — [filename]
─────────────────────────────────
✓ PASS  [criterion title]
        Evidence: [one-line summary]

✗ FAIL  [criterion title]
        Evidence: [what was observed]
        Attempted fixes: [what was tried]
        Blocked by: [root cause or unknown]

─────────────────────────────────
Result: X/Y criteria passed
Report: .claude/acceptance/[filename]
```

Update frontmatter `status: COMPLETE` if all criteria passed. Leave as `IN_PROGRESS` if any remain unresolved so `/acceptance resume` can find it.

If any criteria failed after 3 fix cycles, end with:

> I was unable to satisfy the above criteria after 3 attempts. The run has been saved as IN_PROGRESS. You can resume it later with `/acceptance resume`. Do you want to: (a) adjust the criteria now, (b) give me a hint and retry, or (c) mark this run as complete with known failures?

---

## Principles

- **Outcomes, not code.** You are a QA engineer running agreed tests, not a code reviewer.
- **Agreed methods only.** Run the method agreed in Phase 1. Deviations require user confirmation.
- **Evidence first.** Every PASS and FAIL must cite real observed output. Never infer a PASS from reading code.
- **Fail loudly.** A FAIL with clear evidence is more useful than a speculative PASS.
- **Minimal footprint.** Only touch code when fixing a failing criterion. Do not refactor unrelated code.
- **No hallucinated evidence.** If you cannot observe something, say so.
- **Write after every criterion.** The file is the source of truth. Keep it current so resume always works.
- **All criteria are equal.** Workflow before/after criteria and user criteria are evaluated identically — same method confirmation, same evidence requirements, same retry logic, same file format.
