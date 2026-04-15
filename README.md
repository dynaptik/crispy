# CRISPY Orchestrator

CRISPY (how QRSPI sounds to me phonetically) is a multi-agent orchestration plugin for software development. It implements an 8-phase state machine based on the QRSPI methodology (Question, Research, Structure, Plan, Implement).

Its purpose is to make agentic coding more reliable while also being heavily opinionated what architecture style lends itself to agentic coding (vertical slices or moduliths, alternatively) and how it should be packaged (agent plugin architecture rather than scavenging for .md files all the time).

Note: while some things like the 40 instructions limit seem arbitary, they are grounded in instruction-to-weight-ratios which are often described in the community as limits when models start to "drift". Feel free to question some choices by opening issues to discuss it.

## Core Principles

* **Context Firewalls:** Sub-agents are isolated. The Researcher is physically blinded from the feature task to ensure objective findings
* **Vertical Slices:** Development is broken into independent, testable slices (Logic + API + Tests) rather than horizontal architectural layers
* **Instruction Budgeting:** Each agent operates with fewer than 40 instructions to maintain high reasoning quality
* **Automated Context Flushing:** The orchestrator restarts the implementation agent between every vertical slice to prevent the 'plan-reading illusion.

## The 8 Phases

1. **Question:** Decompose intent into `01_task.md` and `02_questions.md`
2. **Research:** Objective codebase analysis; outputs `03_research.md`
3. **Design:** Architectural alignment; outputs `04_design.md` (Human Gate)
4. **Structure:** Map vertical implementation slices; outputs `05_structure.md`
5. **Plan:** Tactical task list and checkboxes; outputs `06_plan.md`
6. **Worktree:** Isolated git environment creation via MCP for clean execution
7. **Implement:** Execute slices one-by-one, flushing context after each slice
8. **PR:** Assemble the final pull request with a full audit trail

## Installation & Setup

### Requirements
* **VS Code Copilot Agent Plugin:** Preview feature, `chat.plugins.enabled` required
* **MCP**: Model Context Protocol enabled environment
* **Git (2.40+):** Required for `git worktree` isolation
* **WSL2 / Linux / macOS:** Recommended for optimal MCP file-system performance

### Setup
```bash
# Clone the plugin
git clone https://github.com/dynaptik/crispy-plugin

# Install dependencies
cd crispy-plugin
npm install

# Build the plugin
npm run build
```

## Usage

Start the pipeline with a single command:

```bash
/crispy "Implement rate limiting for the /v1/auth endpoint"
```
### Commands
* `/crispy-status`: View current phase and artifact completion
* `/crispy-resume`: Pick up from the last validated checkpoint
* `/approve-design`: Proceed to implementation after reviewing the design

## CRISPY Project Tree

### `crispy/` artifacts
This folder is the Memory of the project. By naming files 01_ through 07_, we ensure that any developer (or a resumed AI agent) can see the logical progression. If the AI hallucinates during implementation, you can point it back to 04_design.md as the ground truth.

### `.crispy-worktree/`
This is the Execution Sandbox. Because your WSL2 Git (2.43.0) supports git worktree, the orchestrator creates this folder to run tests. This is the "Context Firewall" at the file-system level; the agent cannot see your uncommitted local files, ensuring it only builds based on what is in the repository and the artifacts.

### `src/features/` directory
This reflects the Vertical Slice Architecture:
* **Traditional:** You'd have src/controller/authConntroller.ts and src/middleware/rateLimiter.ts
* **Make it CRISPY:** Everything related to the "rate-limiting" feature is co-located. This makes it easier for the AI to "research" the feature later because the context is concentrated in one folder rather than scattered across the whole tree

### `state.json`
This file is what makes the `/crispy resume` command possible. It maps which artifacts have been validated by a human. If you stop at Phase 3 (Design), the `state.json` ensures the agent doesn't try to start implementation without your `/approve-design` command.

```
my-project/
├── .crispy/                        # CRISPY ARTIFACTS (State & Context)
│   └── task-2026-04-15/            # unique folder per feature/task
│       ├── state.json              # phase progress (1-8) tracker
│       ├── 01_task.md              # raw user intent (this is your nucleus!)
│       ├── 02_questions.md         # socratic inquiries (The "Q" Phase)
│       ├── 03_research.md          # blind codebase facts (The "R" Phase)
│       ├── 04_design.md            # approved architecture (The "Brain Surgery")
│       ├── 05_structure.md         # vertical slice definitions (Checkpoints)
│       ├── 06_plan.md              # tactical task-by-task execution list
│       └── 07_implementation.log   # verification results for each slice
│
├── .crispy-worktree/               # ISOLATED EXECUTION (Phase 6-7)
│   └── implementation-sandbox/     # physical Git Worktree (Clean Room)
│       └── [Checked out code]      # where the agent actually writes code
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
├── .gitignore                      # includes .crispy-worktree/ and state.json
└── package.json
```

## The opinionated parts
* Implementation is a .log rather than a .md file since it contains stdout and can easily grow too large. To preserve "instruction budget" I try to avoid digestion by the agent. Logfiles are usually not picked up, except there is an explicit need to debug something.
* When talking about execution sandbox and isolation I mostly talk about keeping the context and "focus" of agents sharp and avoid pollution. Additionally I urge you to use actual sandboxing/isolations like devcontainers, microVMs, bubblewrap/seatbelt or whatever your heart desires. I'm personally a fan of [https://github.com/always-further/nono](https://github.com/always-further/nono) - YTMMV, your threat model may vary.

## Credits
This project is grounded in the research by Dexter Horthy, the articles of Alex Lavaee and the QRSPI prompt engineering work by Matan Shavit. I (Danijel Milicevic - dynaptik) just did some legwork and put it all together.

## License
MIT
