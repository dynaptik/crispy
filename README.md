# CRISPY Orchestrator

CRISPY (how QRSPI sounds to me phonetically) is a multi-agent orchestration plugin for software development. It implements an 8-phase state machine based on the QRSPI methodology (Question, Research, Structure, Plan, Implement).

Its purpose is to make agentic coding more reliable while also being heavily opinionated what architecture style lends itself to agentic coding (vertical slices or moduliths, alternatively) and how it should be packaged (agent plugin architecture rather than scavenging for .md files all the time).

Note: while some things like the 40 instructions limit seem arbitary, they are grounded in instruction-to-weight-ratios which are often described in the community as limits when models start to "drift". Feel free to question some choices by opening issues to discuss it.

## Core Principles

* **Context Firewalls:** Sub-agents are isolated by instruction boundaries. The Researcher is forbidden from reading `01_task.md` via prompt-level enforcement — it only sees `02_questions.md`. This is a logical firewall (the model respects the instruction), not a tool-level restriction.
* **Vertical Slices:** Development is broken into independent, testable slices (Logic + API + Tests) rather than horizontal architectural layers.
* **Instruction Budgeting:** Each agent is authored with fewer than 40 instructions (self-documented via `Budget: N/40` annotations). This is an authoring discipline to stay below empirical drift thresholds, not a runtime enforcement mechanism.
* **Enforced Context Flushing:** The Builder agent has no outbound handoffs and is instructed to exit after each slice. The user triggers the next slice via `/crispy:resume`, which reinvokes the Builder with a clean context window. This is manual-trigger flushing, not an automated loop.

## The 8 Phases

1. **Question:** Decompose intent into `01_task.md` and `02_questions.md`
2. **Research:** Objective codebase analysis; outputs `03_research.md`
3. **Design:** Architectural alignment; outputs `04_design.md` (Human Gate)
4. **Structure:** Map vertical implementation slices; outputs `05_structure.md`
5. **Plan:** Tactical task list and checkboxes; outputs `06_plan.md`
6. **Worktree:** Isolated git environment for clean execution (recommended, not automated)
7. **Implement:** Execute slices one-by-one, flushing context after each slice
8. **PR:** Assemble the final pull request with a full audit trail

## Installation & Setup

### Requirements
* **VS Code** with GitHub Copilot Chat — agent plugins are a preview feature, enable with `chat.plugins.enabled`
* Alternatively: **GitHub Copilot CLI** (full plugin support)

### Setup

No build step required. The plugin is pure markdown and JSON.

```bash
# Option A: Install from source via VS Code Command Palette
# Run "Chat: Install Plugin From Source" and enter:
https://github.com/dynaptik/crispy

# Option B: Clone locally and register via settings
git clone https://github.com/dynaptik/crispy
```

If cloned locally, add it to your VS Code settings:
```jsonc
// settings.json
"chat.pluginLocations": {
    "/path/to/crispy": true
}
```

## Usage

Start the pipeline with the `/crispy:start` skill:

```
/crispy:start "Implement rate limiting for the /v1/auth endpoint"
```

### Skills
* `/crispy:start`: Begin the QRSPI pipeline with a feature description
* `/crispy:status`: View current phase and artifact completion
* `/crispy:resume`: Pick up from the last validated checkpoint, or manually approve the design

### Agents

The plugin provides 6 specialized agents that form the handoff chain:

**Questioner** → **Researcher** → **Architect** → *(human gate: click "Approve Design" button)* → **Structurer** → **Planner** → **Builder**

Each agent has restricted tool access and a scoped instruction budget. The Researcher is explicitly forbidden from reading `01_task.md` (context firewall). The Builder exits after one slice (context flushing).

When the Architect finishes, a clickable **"Approve Design"** button appears in chat. If the button is no longer visible (e.g., session was closed), use `/crispy:resume` instead.

## CRISPY Project Tree

### `crispy/` artifacts
This folder is the Memory of the project. By naming files 01_ through 07_, we ensure that any developer (or a resumed AI agent) can see the logical progression. If the AI hallucinates during implementation, you can point it back to 04_design.md as the ground truth.

### `.crispy-worktree/` (optional)
If you use `git worktree` for slice isolation, this is where worktrees live. The plugin recommends but does not automate worktree creation — you or your CI can set this up. The isolation keeps the agent focused on repository state rather than uncommitted local changes.

### `src/features/` directory
This reflects the Vertical Slice Architecture:
* **Traditional:** You'd have src/controller/authConntroller.ts and src/middleware/rateLimiter.ts
* **Make it CRISPY:** Everything related to the "rate-limiting" feature is co-located. This makes it easier for the AI to "research" the feature later because the context is concentrated in one folder rather than scattered across the whole tree

### Phase ordering
The handoff chain between agents enforces phase ordering declaratively. Each agent hands off to the next with `send: false`, meaning the user must explicitly trigger the transition. The human gate after Phase 3 (Design) is enforced by the `/crispy-approve` skill — without it, the Structurer is never invoked.

```
my-project/
├── .crispy/                        # CRISPY ARTIFACTS (State & Context)
│   ├── 01_task.md                  # raw user intent (this is your nucleus!)
│   ├── 02_questions.md             # socratic inquiries (The "Q" Phase)
│   ├── 03_research.md              # blind codebase facts (The "R" Phase)
│   ├── 04_design.md                # approved architecture (The "Brain Surgery")
│   ├── 05_structure.md             # vertical slice definitions (Checkpoints)
│   ├── 06_plan.md                  # tactical task-by-task execution list
│   └── 07_implementation.log       # verification results for each slice
│
├── src/                            # PROJECT SOURCE (Vertical Slice Style)
│   └── features/                   # core business logic grouping
│       └── auth-rate-limiting/     # a completed CRISPY vertical slice (ts-example)
│           ├── middleware.ts       # route protection logic
│           ├── redis-store.ts      # storage implementation
│           ├── types.d.ts          # feature-specific types
│           └── __tests__/          # integrated slice tests
│               └── rate-limit.test.ts
│
└── .gitignore
```

## The opinionated parts
* Implementation is a .log rather than a .md file since it contains stdout and can easily grow too large. To preserve "instruction budget" I try to avoid digestion by the agent. Logfiles are usually not picked up, except there is an explicit need to debug something.
* When talking about execution sandbox and isolation I mostly talk about keeping the context and "focus" of agents sharp and avoid pollution. Additionally I urge you to use actual sandboxing/isolations like devcontainers, microVMs, bubblewrap/seatbelt or whatever your heart desires. I'm personally a fan of [https://github.com/always-further/nono](https://github.com/always-further/nono) - YTMMV, your threat model may vary.

## Credits
This project is grounded in the research by Dexter Horthy, the articles of Alex Lavaee and the QRSPI prompt engineering work by Matan Shavit. I (Danijel Milicevic - dynaptik) just did some legwork and put it all together.

## License
MIT
