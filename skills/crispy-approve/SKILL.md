---
name: crispy-approve
description: "Approve the design in 04_design.md and proceed to vertical slice structuring, planning, and implementation. This is the human gate in the QRSPI workflow."
argument-hint: "Optional: notes or constraints for the implementation phase"
---

# CRISPY Approve Design

Signals that the architecture and design artifacts in Phase 04 are satisfactory. Invoking this command moves the orchestrator into Phase 05 (Structuring) and Phase 06 (Building) within the isolated worktree.

## Prerequisites

- Phase D (Design) must be complete
- `04_design.md` must exist in `.crispy/`

## What happens

1. Validates that the current phase is Design
2. Invokes the **Structurer** agent to break the design into vertical slices (`05_structure.md`)
3. Invokes the **Planner** agent to create tactical checklists (`06_plan.md`)
4. Initializes the Git Worktree for isolated implementation
5. Begins the **Builder** loop — one slice at a time with context flushing between slices
