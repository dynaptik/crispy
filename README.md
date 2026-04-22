<!-- LOGO -->
<h1 align="center">
    <img src="https://github.com/user-attachments/assets/45ed2618-e450-4c49-b281-a79edacd8876" alt="CRISPY logo" width="512">
    <br />
    CRISPY Orchestrator
</h1>

<p align="center">
    Opinionated multi-agent orchestration plugin for software development.
    <br />
    QRSPI-inspired staged workflow for GitHub Copilot in VS Code.
    <br />
    <a href="#core-principles">About</a>
    ·
    <a href="#installation--setup">Install</a>
    ·
    <a href="#usage">Usage</a>
    ·
    <a href="#the-8-phases">Workflow</a>
    ·
    <a href="#license">License</a>
</p>

CRISPY (how QRSPI sounds to me phonetically) is a multi-agent orchestration plugin for software development. It implements an opinionated staged workflow around the QRSPI methodology (Question, Research, Structure, Plan, Implement).

Its purpose is to make agentic coding more reliable while also being heavily opinionated what architecture style lends itself to agentic coding (vertical slices or moduliths, alternatively) and how it should be packaged (agent plugin architecture rather than scavenging for .md files all the time).

Note: while some things like the 40 instructions limit seem arbitary, they are grounded in instruction-to-weight-ratios which are often described in the community as limits when models start to "drift". Feel free to question some choices by opening issues to discuss it.

## Core Principles

* **Context Firewalls:** Sub-agents are isolated by instruction boundaries. The Researcher is forbidden from reading `01_task.md` via prompt-level enforcement — it only sees `02_questions.md`. This is a logical firewall (the model respects the instruction), not a tool-level restriction.
* **Vertical Slices:** Development is broken into independent, testable slices that deliver one narrow capability end-to-end (logic + interface + tests) rather than horizontal architectural layers. This is a planning and execution rule, not a guarantee that every repo will use a `src/features/` folder scheme.
* **Instruction Budgeting:** Each agent is authored with fewer than 40 instructions (self-documented via `Budget: N/40` annotations). This is an authoring discipline to stay below empirical drift thresholds, not a runtime enforcement mechanism.
* **Enforced Context Flushing:** The Builder agent has no outbound handoffs and is instructed to exit after each slice. The user triggers the next slice via `/crispy:resume`, which reinvokes the Builder with a clean context window. This is manual-trigger flushing, not an automated loop.

## The 8 Phases

1. **Question:** Decompose intent, collect any missing user-owned constraints, and write `02_questions.md`
2. **Research:** Objective codebase analysis; outputs `03_research.md`
3. **Design:** Architectural alignment; outputs `04_design.md` (Human Gate)
4. **Structure:** Map vertical implementation slices; outputs `05_structure.md`
5. **Plan:** Tactical task list and checkboxes; outputs `06_plan.md`
6. **Repository Setup:** Optionally initialize git, then optionally create a worktree for isolated execution
7. **Implement:** Execute slices one-by-one, flushing context after each slice
8. **Wrap Up:** Write `08_changelog.md` with a concise implementation summary and merge status

## Installation & Setup

### Requirements
* **VS Code** with GitHub Copilot Chat — agent plugins are a preview feature, enable with `chat.plugins.enabled`
* **Git** — optional, but recommended for commits, worktrees, and clean iteration history. CRISPY can still run without git if you choose not to initialize a repository.

### Setup

No build step required. The plugin is pure markdown and JSON.

Git is not required to install the plugin if you use the VS Code "Install Plugin From Source" flow. It is only needed for the clone-based setup path below and for git-backed CRISPY workflow features.

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
    "/local/path/to/crispy": true
}
```

## Usage

Start the pipeline with the `/crispy:start` skill:

```
/crispy:start "Implement rate limiting for the /v1/auth endpoint"
```

### Skills
* `/crispy:start`: Begin the QRSPI pipeline with a feature description
* `/crispy:status`: View artifact state, likely current phase, worktree mode, and the next expected action
* `/crispy:resume`: Pick up from the last validated checkpoint, or continue after an approved design

### Agents

The plugin provides 6 specialized agents that form the handoff chain:

**Questioner** → **Researcher** → **Architect** → *(human gate: Architect asks for approval, then the user explicitly continues)* → **Structurer** → **Planner** → **Builder**

Each agent has restricted tool access and a scoped instruction budget. The Researcher is explicitly forbidden from reading `01_task.md` (context firewall). The Builder exits after one slice (context flushing).

Phase Q may ask the user a small number of structured clarification questions when core requirements are still missing. Those answers are treated as confirmed inputs. `02_questions.md` is for the Researcher, not a questionnaire the user is expected to answer manually.

When the Architect finishes `04_design.md`, it stops and presents a single approval gate: reply with revision instructions if changes are needed, or continue the workflow with `/crispy:resume`.

## CRISPY Project Tree

### `.crispy/` working state
This folder is ephemeral scratch space for the current pipeline run. Each `/crispy:start` overwrites the previous artifacts. By naming files 01_ through 08_, any developer (or a resumed AI agent) can see the logical progression of the *current* feature. If the AI hallucinates during implementation, you can point it back to 04_design.md as the ground truth.

Once the iteration is concluded, these files have served their purpose — the decision history is preserved in your git log if you are using git, plus the final `08_changelog.md` summary. If you want to keep them, commit before starting a new run.

### `.crispy-worktree/` (optional)
If the project is not yet a git repo, the Planner asks whether it should initialize one first. Only after the repo is confirmed or initialized does it offer `.crispy-worktree/` as an isolated checkout. On a new `/crispy:start` run, any existing worktree is cleaned up automatically. The Builder works inside the worktree to keep the main branch clean during implementation.

### Physical package layout
CRISPY does not mandate a single folder layout. The important invariant is that slices are planned and implemented end-to-end; the physical packaging style is an explicit design choice.

* **Existing codebase:** Preserve the dominant package boundaries discovered in Research and confirmed in Design unless there is a deliberate, justified reason to change them.
* **Greenfield or new subsystem:** Design must explicitly choose a packaging style up front (for example: co-located feature folders, a stage-centric modulith, or plugin/module packages) and justify why it fits the repo.
* **`src/features/` is one valid expression, not the definition of CRISPY.**

Illustrative greenfield TypeScript example:

### Phase ordering
The handoff chain between agents enforces phase ordering declaratively. Each agent hands off to the next with `send: false`, meaning the user must explicitly trigger the transition. The human gate after Phase 3 (Design) is enforced by the Architect asking for approval plus the explicit handoff trigger — without approval, the Structurer is never invoked.

```
my-typescript-service/
├── .crispy/                        # CRISPY ARTIFACTS (State & Context)
│   ├── 01_task.md                  # raw user intent (this is your nucleus!)
│   ├── 02_questions.md             # socratic inquiries (The "Q" Phase)
│   ├── 03_research.md              # blind codebase facts (The "R" Phase)
│   ├── 04_design.md                # approved architecture (The "Brain Surgery")
│   ├── 05_structure.md             # vertical slice definitions (Checkpoints)
│   ├── 06_plan.md                  # tactical task-by-task execution list
│   ├── 07_implementation.log       # verification results for each slice
│   └── 08_changelog.md             # concise run summary and merge status
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
This project is grounded in the research by Dexter Horthy, the articles of Alex Lavaee and the QRSPI prompt engineering work by Matan Shavit. I (Danijel Milicevic - dynaptik) just did some legwork and put it all together, simplify some things and test it across multiple personal and professional projects (which is ongoing).

## License
MIT
