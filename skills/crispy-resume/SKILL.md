---
name: crispy-resume
description: "Resume CRISPY from the last validated checkpoint. Use when a session was interrupted or when switching workstations."
---

# CRISPY Resume

Reloads the state machine from `.crispy/state.json`. Use this if the session was interrupted or if you are moving between workstations (e.g., WSL2 to Mac) and need to pick up where the orchestrator left off.

## What it does

1. Reads `state.json` to determine the last validated phase
2. Verifies that the expected artifacts for completed phases exist
3. Resumes from the next pending phase
4. If in the implementation phase, identifies which vertical slice was last completed and continues from the next one
