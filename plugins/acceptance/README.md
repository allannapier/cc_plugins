# acceptance plugin

Interactive acceptance testing for Claude Code projects.

## What it does

The `/acceptance` skill guides you through a structured QA session:

1. **Gather criteria** — describe what should be true; Claude rewrites each as a specific, observable outcome
2. **Agree evaluation methods** — confirm exactly how each criterion will be tested before any verification runs
3. **Verify outcomes** — Claude runs the agreed tests, records real evidence, and marks each PASS or FAIL
4. **Save results** — all criteria and evidence are written to `.claude/acceptance/` so runs can be resumed if interrupted

## Installation

```bash
# From your project directory
claude plugin install https://github.com/allannapier/cc_plugins/tree/main/plugins/acceptance
```

## Usage

```
/acceptance                         # Start a new session
/acceptance my first criterion      # Start with a criterion pre-filled
/acceptance resume                  # Resume an interrupted run
```

## Skills

| Skill | Trigger | Description |
|---|---|---|
| `acceptance` | `/acceptance` | Run an acceptance verification session |

## Example session

```
/acceptance the login form shows errors on empty submit

Claude rewrites: "Submitting the login form with all fields empty shows
inline validation errors on each required field without calling the API"

Method: Open form in Chrome, submit empty, screenshot errors, check Network tab.

Does this look right? > yes

What's the next criterion? > done

[runs verification, records evidence, marks PASS/FAIL]
```
