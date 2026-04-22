---
name: start
description: "Start the CRISPY workflow. Use when you want to build a new feature or vertical slice using the QRSPI methodology."
argument-hint: "Describe the feature or vertical slice you want to build"
---

# CRISPY Start

Initiates the CRISPY Orchestrator. This command sets up the project structure, activates the context firewalls, and begins Phase 01 (Questioning) to define the scope of the new vertical slice.

## What happens

1. Clean up previous run state using `execute`:
   - `git worktree remove .crispy-worktree --force 2>/dev/null; git branch -D crispy/implementation 2>/dev/null; rm -f .crispy/*.md .crispy/*.log`
   - This is a single command. Errors from missing worktrees/branches/files are suppressed.
2. Create the file `.crispy/01_task.md` with the user's intent
3. Hand off to the **CRISPY Questioner** subagent to collect any missing user constraints via structured questions when needed, then write targeted questions for the Researcher
4. The handoff chain continues through Research and Design automatically
5. Stops at the **human gate** — after writing `04_design.md`, the Architect tells the user to either send revision instructions or continue with `/crispy:resume`.

## Important

- Do NOT read the plugin's own `agents/` or `skills/` directories. Subagents are invoked by name — VS Code loads their instructions automatically.
- Do NOT verify or list the `.crispy/` directory after creating files in it.

## Usage

Provide a clear description of the feature or change you want to implement:

```
/crispy:start Implement rate limiting for the /v1/auth endpoint
```
