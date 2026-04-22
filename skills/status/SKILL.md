---
name: status
description: "Show the current CRISPY artifact state and likely phase. Use to check which artifacts exist, whether the design gate is pending, whether a worktree is configured, and what the next likely action is."
---

# CRISPY Status

Displays a short artifact-based summary of the current CRISPY run. Use only the files already written under `.crispy/` and the metadata at the top of `06_plan.md`.

## What to do

1. Check which of these artifacts exist: `01_task.md` through `08_changelog.md`.
2. Treat the highest-numbered existing artifact as the best current checkpoint.
3. If `06_plan.md` exists, read only its metadata block at the top and report whether a worktree working directory is configured.
4. If `07_implementation.log` exists, read only the headings plus any `## Wrap Up` section to determine whether implementation is in progress or all slices appear complete.
5. If `08_changelog.md` exists, treat the run as concluded and report the merge status recorded there.

## What it should report

- Existing artifacts (`01_task.md` through `08_changelog.md`)
- Likely current phase or concluded state based on the highest artifact present
- Whether the design approval gate is still pending
- Whether a worktree is configured in `06_plan.md`
- The next likely action, for example:
	- no artifacts: start with `/crispy:start`
	- `04_design.md` is the latest artifact: send revision instructions or continue with `/crispy:resume`
	- `06_plan.md` is the latest artifact: start implementation
	- `07_implementation.log` is the latest artifact: resume the next slice or finish wrap-up
	- `08_changelog.md` exists: the current run is concluded

## What it must not claim

- Do not claim to detect context firewall violations.
- Do not claim to know merge state unless `08_changelog.md` says so.
- Do not infer progress from chat history, git history, or unstated assumptions.
