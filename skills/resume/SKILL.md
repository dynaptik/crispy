---
name: resume
description: "Resume CRISPY from the last validated checkpoint. Use when a session was interrupted, when switching workstations, or when the design was approved and you want to continue the workflow."
---

# CRISPY Resume

Use this if the session was interrupted, you are switching workstations, or the design was approved and you want to continue from the current checkpoint.

## What it does

1. Scans `.crispy/` for existing artifacts (`01_task.md` through `07_implementation.log`)
2. Determines the last completed phase from the highest-numbered artifact present
3. Invokes the next agent in the handoff chain:
   - `02_questions.md` exists → invoke **Researcher**
   - `03_research.md` exists → invoke **Architect**
   - `04_design.md` exists → invoke **Structurer** (this is the manual design approval path)
   - `05_structure.md` exists → invoke **Planner**
   - `06_plan.md` exists → invoke **Builder**
4. If in the implementation phase, identifies which vertical slice was last completed and continues from the next one
