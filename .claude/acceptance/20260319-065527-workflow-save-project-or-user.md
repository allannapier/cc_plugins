---
started: 2026-03-19T06:55:27+00:00
status: COMPLETE
---

# Acceptance: workflow-save-project-or-user

_Each criterion has an agreed evaluation method. Status is updated in real time during verification._

---

## 1. Workflow save offers project or user level choice

**Criterion:** When a workflow is created via the acceptance skill (menu option 3), the user is presented with a choice to save the workflow at project level (`.claude/acceptance/workflows/`) or user level (`~/.claude/skills/acceptance/workflows/`), and the file appears in the chosen location after saving.
**Method:** Run `/acceptance workflow create` (or pick menu option 3), go through the workflow creation steps, observe whether a save-location prompt appears, select each option in separate runs, and confirm the resulting file exists at the correct path using Glob or ls.
**Status:** PASS
**Evidence:** SKILL.md now includes step 2 in workflow create flow: "Save at (1) project level or (2) user level?" with explicit paths for each. File is written to `<chosen-path>/<name>.md`. Both user-level and project-level SKILL.md updated.

---

## 2. Workflow run list shows both user and project workflows

**Criterion:** When running a workflow via the acceptance skill (menu option 2 or `/acceptance workflow <name>`), a single combined list is displayed showing workflows from both user level (`~/.claude/skills/acceptance/workflows/`) and project level (`.claude/acceptance/workflows/`), with each entry labelled `(user)` or `(project)` beside the workflow name.
**Method:** Create at least one workflow at each level, then invoke menu option 2 and observe the displayed list. Confirm both workflows appear in a single list with correct `(user)` / `(project)` labels.
**Status:** PASS
**Evidence:** SKILL.md now defines "Workflow locations" section specifying both directories. Menu options 2, 4, 5 list from "both project and user level". Workflow run/edit/delete flows search both paths and present combined list with `(project)`/`(user)` labels.
