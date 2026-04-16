---
name: start
description: "Start the CRISPY 8-phase implementation pipeline. Use when you want to build a new feature or vertical slice using the QRSPI methodology — Question, Research, Design, Structure, Plan, Worktree, Implement, PR."
argument-hint: "Describe the feature or vertical slice you want to build"
---

# CRISPY Start

Initiates the CRISPY Orchestrator. This command sets up the project structure, activates the context firewalls, and begins Phase 01 (Questioning) to define the scope of the new vertical slice.

## What happens

1. Create the file `.crispy/01_task.md` with the user's intent (this implicitly creates the directory)
2. Hand off to the **CRISPY Questioner** subagent to decompose the intent into 5-10 targeted technical questions
3. The handoff chain continues through Research and Design automatically
4. Stops at the **human gate** — the user must review `04_design.md` and run `/crispy:resume` to continue

## Important

- Do NOT read the plugin's own `agents/` or `skills/` directories. Subagents are invoked by name — VS Code loads their instructions automatically.
- Do NOT verify or list the `.crispy/` directory after creating files in it.

## Usage

Provide a clear description of the feature or change you want to implement:

```
/crispy-start Implement rate limiting for the /v1/auth endpoint
```
