---
name: crispy-status
description: "Show the current CRISPY phase, completed artifacts, and any pending blockers. Use to check progress of the QRSPI workflow."
---

# CRISPY Status

Displays a high-level summary of the current CRISPY session, including the active phase, completed artifacts, and any pending blockers or context firewall violations.

## What it shows

- Current phase (1-8) and phase name
- Which artifacts exist (`01_task.md` through `07_implementation.log`)
- Whether the human gate (design approval) has been passed
- Active vertical slice (if in implementation phase)
- Any context firewall violations detected
