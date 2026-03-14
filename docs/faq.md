# Frequently Asked Questions

This page answers the most common questions about Ouro Loop, covering what it does, how it works, when to use it, and how it compares to other approaches for AI-assisted coding. Each answer is self-contained and references the relevant documentation for deeper exploration.

---

## General

### What is Ouro Loop?

Ouro Loop is an open-source framework that gives AI coding agents (Claude Code, Cursor, Aider, Codex) a structured autonomous loop with runtime-enforced guardrails. It implements bounded autonomy: the developer defines absolute constraints (DANGER ZONES, NEVER DO rules, IRON LAWS) using the BOUND system, then the agent loops autonomously through Build, Verify, and Self-Fix cycles. When verification fails, the agent consults its remediation playbook, reverts, tries a different approach, and reports what it did.

### What problem does Ouro Loop solve?

In the era of "vibe coding," unbound AI agents hallucinate file paths, break production constraints, regress architectural patterns, and get stuck in infinite fix-break loops. The common solution — pausing to ask humans — negates the promise of autonomous coding. Ouro Loop solves this by establishing formal constraints before any code is written, then enforcing those constraints at the runtime level while the agent operates autonomously.

### How is this different from just using `.cursorrules` or `CLAUDE.md`?

`.cursorrules` and `CLAUDE.md` define static instructions that the agent can ignore. Ouro Loop adds a runtime loop — state tracking, multi-layer verification, autonomous remediation, and phase management. The agent does not just follow rules; it verifies compliance, detects drift, and self-corrects. Most importantly, BOUND constraints are enforced by runtime hooks (exit 2 hard-block), not by the agent's good behavior. See the [Comparison page](comparison.md) for a detailed breakdown.

### What is bounded autonomy in AI coding?

Bounded autonomy is a paradigm where AI coding agents are granted full autonomous decision-making power within explicitly defined constraints. By defining the 20 things an agent cannot do (the BOUND), you implicitly authorize it to do the 10,000 things required to solve the problem. It is the middle path between human-in-the-loop (constant interruptions) and unbounded agents (unconstrained risk). See [Core Concepts](concepts.md#bounded-autonomy) for the full definition.

---

## Setup and Usage

### Do I need to install anything?

No. Zero dependencies. Pure Python 3.10+ standard library. Clone the repo and point your agent at `program.md`. Optionally install Claude Code hooks for runtime enforcement. See the [Quick Start Guide](guides/quick-start.md).

### What AI agents does Ouro Loop work with?

Ouro Loop is agent-agnostic. It works with any AI coding assistant that can read files and execute terminal commands:

- **Claude Code** — Native `program.md` skill support + 4 enforcement hooks
- **Cursor** — Use `.cursorrules` to reference Ouro Loop modules
- **Aider** — Terminal-based AI pair programmer
- **Codex CLI** — OpenAI's coding agent
- **Windsurf** — Codeium's AI IDE

See the [Claude Code Integration Guide](guides/claude-code.md) and [Cursor Integration Guide](guides/cursor.md) for agent-specific setup.

### How do I define a BOUND?

The BOUND is defined in your project's `CLAUDE.md` file with three sections:

- **DANGER ZONES** — Files or directories where incorrect changes cause catastrophic failure. List each file path with a brief explanation of why it is dangerous.
- **NEVER DO** — Absolute prohibitions that apply across the entire codebase. These are invariant rules that no task justifies breaking.
- **IRON LAWS** — Measurable invariants that must always be true after any change. Every IRON LAW should be verifiable programmatically.

See [Examples](examples/index.md) for real-world BOUND definitions from blockchain, financial, consumer product, and ML research projects.

### How do I add guardrails to Claude Code?

Ouro Loop provides 4 Claude Code Hooks that enforce constraints at the tool level. Install them by copying `hooks/settings.json.template` to `.claude/settings.json` and editing the paths. The `bound-guard.sh` hook parses your CLAUDE.md DANGER ZONES and physically blocks edits to protected files. No agent can bypass exit code 2. See the [Claude Code Integration Guide](guides/claude-code.md) for step-by-step setup.

### How do I let an AI agent code overnight?

Define your BOUND (DANGER ZONES, NEVER DO, IRON LAWS) in CLAUDE.md, install the hooks, then tell the agent to read `program.md` and start the loop. The agent will iterate through Build, Verify, and Self-Fix cycles autonomously. When verification fails, it remediates and retries. When it passes, it advances to the next phase. The loop continues until all phases are complete.

---

## How It Works

### Can the agent really fix its own mistakes?

Yes, within BOUND. When verification fails and the issue is inside the boundary (not a DANGER ZONE), the agent consults `modules/remediation.md` for a decision playbook: revert, retry with a different approach, or escalate. It reports what it did, not what it is thinking of doing. In a real blockchain session, the agent autonomously remediated 4 failures across 5 hypotheses and found a root cause that was architectural (HTTP routing), not code-level. See the [Blockchain Session Log](session-logs/blockchain-l1.md) for the full record.

### What are the five verification gates?

Ouro Loop uses a multi-layer verification system with five gates:

| Gate | Checks | Prevents |
|------|--------|----------|
| **EXIST** | Do referenced files, APIs, and modules actually exist? | Hallucination |
| **RELEVANCE** | Is current work related to the original task? | Scope drift |
| **ROOT_CAUSE** | Is this fixing the cause, not just a symptom? | Stuck loops |
| **RECALL** | Can the agent still recall key constraints? | Context decay |
| **MOMENTUM** | Is meaningful progress being made? | Velocity death |

Failed gates trigger autonomous remediation. See [Core Concepts](concepts.md#five-verification-gates) for details.

### How to prevent AI agents from hallucinating file paths?

Ouro Loop's EXIST verification gate checks whether referenced files, APIs, and modules actually exist before the agent proceeds. The `bound-guard.sh` hook also validates file paths against the project structure. If a file does not exist, the gate fails and triggers autonomous remediation — the agent corrects its reference instead of proceeding with hallucinated paths.

### How to prevent context decay in long AI coding sessions?

Ouro Loop addresses context decay through the RECALL verification gate and the `recall-gate.sh` hook. The gate monitors whether the agent can still recall key constraints. The hook fires before context compression (PreCompact event) and re-injects the BOUND section into the compressed context, preventing constraint amnesia during long sessions.

### Can AI agents fix their own bugs autonomously?

Yes — this is called autonomous remediation. When a verification gate fails, the agent does not alert a human. It reads its error logs, consults its remediation playbook, and takes action: revert, retry with a different approach, or escalate only if a DANGER ZONE is involved. The key constraint is BOUND — the agent can self-fix anything inside the boundary. See [Core Concepts](concepts.md#autonomous-remediation) for the full explanation.

### How long can the agent run autonomously?

As long as phases remain. Each phase is independently verifiable, so the agent can run for hours across many phases. The NEVER STOP instruction in `program.md` keeps the loop going until all phases pass or an EMERGENCY-level issue is hit.

---

## When to Use (and When Not To)

### When should I use Ouro Loop?

Ouro Loop is designed for scenarios where you need an AI agent to work autonomously for extended periods without human babysitting:

- **Overnight autonomous development** — Define BOUND, start the agent, sleep. Wake up to a log of phases completed and verified.
- **Long-running refactoring** — Let the agent refactor a large codebase in phases, with verification gates ensuring nothing breaks.
- **Multi-phase feature development** — Break complex features into severity-ordered phases. The agent handles CRITICAL changes first, then HIGH, then MEDIUM.
- **Production-safe AI coding** — For financial systems, blockchain infrastructure, and any domain where "move fast and break things" is unacceptable.

### When should I NOT use Ouro Loop?

Do not use Ouro Loop for:

- **Quick prototypes or hackathon projects** — The BOUND setup overhead is not worth it for throwaway code.
- **Single-file scripts** — The methodology overhead exceeds the benefit.
- **Real-time interactive coding** — Ouro Loop is designed for "set it and let it run," not conversational back-and-forth coding.

---

## Architecture and Design

### What are the three files that matter?

- **`program.md`** — The methodology instructions. Defines the six-stage loop (BOUND, MAP, PLAN, BUILD, VERIFY, LOOP) and the autonomous remediation rules. This file is iterated on by the human.
- **`framework.py`** — Lightweight runtime for state tracking, verification gates, and logging. The agent uses this CLI to check its own work. This file can be extended by the agent.
- **`prepare.py`** — Project scanning and initialization. Creates the `.ouro/` state directory. Not modified during operation.

### How does runtime enforcement work?

Ouro Loop includes 4 Claude Code Hooks that enforce BOUND at the tool level:

| Hook | Event | What it does |
|------|-------|-------------|
| `bound-guard.sh` | PreToolUse:Edit/Write | Parses DANGER ZONES, blocks edits to protected files (exit 2) |
| `root-cause-tracker.sh` | PostToolUse:Edit/Write | Tracks per-file edit count, warns at 3+, strongly warns at 5+ |
| `drift-detector.sh` | PreToolUse:Edit/Write | Warns when touching 5+ directories (scope drift) |
| `recall-gate.sh` | PreCompact | Re-injects BOUND into context before compression |

These hooks use exit code 2 to hard-block operations, making violations physically impossible rather than merely inadvisable.

### How is Ouro Loop related to autoresearch?

Ouro Loop was directly inspired by [karpathy/autoresearch](https://github.com/karpathy/autoresearch). Both share the same core idea: give an AI agent a loop and let it iterate autonomously. autoresearch pioneered this for ML training experiments (single metric: val_bpb, single file: train.py, fixed budget: 5 minutes). Ouro Loop generalizes the paradigm to all software engineering with multi-file projects, multi-layer verification, and formal constraint enforcement. See the [Comparison page](comparison.md#ouro-loop-vs-autoresearch) for the detailed side-by-side.
