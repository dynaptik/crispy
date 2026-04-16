---
name: crispy-start
description: "Start the CRISPY 8-phase implementation pipeline. Use when you want to build a new feature or vertical slice using the QRSPI methodology — Question, Research, Design, Structure, Plan, Worktree, Implement, PR."
argument-hint: "Describe the feature or vertical slice you want to build"
---

# CRISPY Start

Initiates the CRISPY Orchestrator. This command sets up the project structure, activates the context firewalls, and begins Phase 01 (Questioning) to define the scope of the new vertical slice.

## What happens

1. Creates the `.crispy/` artifact directory for this task
2. Saves your intent to `01_task.md`
3. Invokes the **Questioner** agent to decompose your intent into 5-10 targeted technical questions
4. Proceeds through Research (with context firewall — Researcher cannot see your intent) and Design
5. Stops at the **human gate** — you must review `04_design.md` and run `/crispy-approve` to continue

## Usage

Provide a clear description of the feature or change you want to implement:

```
/crispy-start Implement rate limiting for the /v1/auth endpoint
```
