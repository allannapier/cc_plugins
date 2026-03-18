---
name: acceptance
description: Gather acceptance criteria interactively, agree evaluation methods with the user, store them in .claude/acceptance/, then verify each criterion and update the file with evidence. Supports resuming interrupted runs.
argument-hint: [criterion] | resume
disable-model-invocation: true
---

You are running an acceptance verification session. Your job is to evaluate **outcomes** — observable behaviour of the running system — not code quality, structure, or style. You may invoke linters or quality tools only when a criterion explicitly requires it (e.g. "the build must pass CI"). Never make judgements about code quality as an end in itself.

---

## Entry point — read this first, follow only one path

**IMPORTANT: these are two completely separate flows. Execute only the one that matches. Never scan for existing runs during a new session.**

### If `$ARGUMENTS` is exactly `resume` or starts with `resume`:
→ Stop here. Go directly to **Resume flow** below. Do not gather criteria. Do not create a new file.

### If `$ARGUMENTS` is anything else (including empty):
→ Stop here. Go directly to **New session flow**. Do not scan `.claude/acceptance/`. Do not list existing runs.

---

## Resume flow — only reached via `/acceptance resume`

1. Ensure `.claude/acceptance/` exists. If the directory is missing or empty, tell the user there are no saved runs and offer to start a new one.

2. Scan all `.md` files in `.claude/acceptance/`. For each file, read the frontmatter `status` field and count criteria by their Status values.

3. List only files where `status: IN_PROGRESS` (i.e. at least one criterion is still PENDING). Format:

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
    → Or install chrome-cli: brew install chrome-cli / see https://github.com/nickcoutsos/chrome-cli
  ✗ psql           — NOT FOUND
    → No database client found in PATH
    → sqlite3 also not available
    → Recommendation: install psql (brew install postgresql) or confirm DB access method
```

For each missing tool, provide:
1. Whether a usable alternative was found in the project
2. An install recommendation if no alternative exists
3. A specific question: "Shall I use [alternative] instead, or do you want to install [tool] first?"

**Do not proceed to Phase 4 until every required tool is either confirmed available or the user has approved an alternative.** If a gap cannot be resolved, tell the user which criteria cannot be verified and ask whether to skip them, defer them, or halt the run entirely.

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
- If all criteria are now resolved, also update the frontmatter `status:` to `COMPLETE`

This ensures the file always reflects current state — if the session ends abruptly, progress is not lost.

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
