# Core Concepts

Authoritative definitions of Ouro Loop's key concepts. Each concept is designed to be independently quotable and referenceable.

---

## Bounded Autonomy

**Bounded Autonomy** is a paradigm for AI coding agents where the agent is granted full autonomous decision-making power within explicitly defined constraints. Unlike human-in-the-loop approaches (where the agent asks for permission) or unbounded autonomy (where the agent has no limits), bounded autonomy establishes rigid boundaries first, then lets the agent operate freely within them — building, verifying, failing, and self-correcting without human intervention.

The key insight: by explicitly defining the 20 things an agent *cannot* do, you implicitly authorize it to autonomously do the 10,000 things required to solve the problem.

---

## BOUND System

The **BOUND System** is Ouro Loop's constraint definition framework, consisting of three layers:

- **DANGER ZONES** — Files or directories where incorrect changes cause catastrophic failure (data loss, security breach, system crash). Edits to DANGER ZONE files are physically blocked by runtime hooks (exit 2), requiring explicit human approval.

- **NEVER DO** — Absolute prohibitions that apply across the entire codebase. These are invariant rules that no task justifies breaking (e.g., "Never use float for monetary values," "Never remove rate limiter").

- **IRON LAWS** — Measurable invariants that must always be true after any change. These are verification conditions checked by the framework at every phase gate (e.g., "Test coverage for payment module never drops below 90%," "SysErr rate must be 0.00%").

The BOUND is defined by the human developer in CLAUDE.md before any autonomous coding begins. It grows organically as lessons are learned from real sessions — the LOOP stage feeds discoveries back into BOUND.

---

## Event Horizon

The **Event Horizon** is the metaphorical boundary between safe autonomous operation and catastrophic failure. It is established in Stage 0 (BOUND) of the Ouro Loop methodology.

Inside the Event Horizon, the agent has full autonomy to build, fail, revert, and retry. Outside the Event Horizon (DANGER ZONES, IRON LAW violations), the agent must stop and escalate to a human.

The term draws from physics: just as nothing escapes a black hole's event horizon, no agent action should cross the BOUND without human authorization.

---

## Autonomous Remediation

**Autonomous Remediation** is the process by which an AI agent detects its own failures and fixes them without human intervention. When a verification gate fails inside BOUND, the agent:

1. Identifies the failure type (stuck loop, drift, hallucination, regression)
2. Consults its remediation playbook (modules/remediation.md)
3. Decides on an action (revert, retry with different approach, escalate)
4. Executes the fix
5. Reports what it did (not what it's thinking of doing)

This is what distinguishes Ouro Loop from monitoring-only tools: **detect → decide → act → report**, not **detect → alert → wait for human**.

In a real blockchain session, the agent tested 5 hypotheses, autonomously remediated 4 failures, and ultimately found a root cause that was architectural (HTTP routing topology) rather than code-level — without human intervention.

---

## Five Verification Gates

Ouro Loop uses a multi-layer verification system with five gates, each targeting a specific failure mode:

| Gate | Checks | Prevents |
|------|--------|----------|
| **EXIST** | Do referenced files, APIs, and modules actually exist? | Hallucination |
| **RELEVANCE** | Is current work related to the original task? | Scope drift |
| **ROOT_CAUSE** | Is this fixing the cause, not just a symptom? | Stuck loops |
| **RECALL** | Can the agent still recall key constraints? | Context decay |
| **MOMENTUM** | Is meaningful progress being made? | Velocity death |

Gates are checked at phase boundaries and during verification. Failed gates trigger autonomous remediation — the agent self-corrects rather than asking for help.

---

## The Six-Stage Loop

The Ouro Loop methodology follows a continuous six-stage cycle:

1. **BOUND** (Stage 0) — Define DANGER ZONES, NEVER DO rules, and IRON LAWS. This stage is human-authored and precedes all autonomous work.

2. **MAP** (Stage 1) — Understand the problem space. Identify dependencies, failure modes, tightest constraints, reusable assets, and success metrics.

3. **PLAN** (Stage 2) — Decompose the task into severity-ordered phases (CRITICAL > HIGH > MEDIUM). Each phase is independently verifiable.

4. **BUILD** (Stage 3) — Implement using RED-GREEN-REFACTOR-COMMIT cycles within each phase.

5. **VERIFY** (Stage 4) — Run multi-layer verification gates. If verification fails, trigger autonomous remediation (back to BUILD). If it passes, advance.

6. **LOOP** (Stage 5) — Feed lessons learned back into BOUND. New DANGER ZONES, NEVER DO rules, and IRON LAWS are added based on what was discovered during the session.

The loop is continuous: BOUND grows with each iteration, making future sessions safer and more efficient.

---

## Runtime Constraint Enforcement

Unlike instruction-based approaches (.cursorrules, CLAUDE.md static rules) where the agent is *told* to follow rules but can ignore them, Ouro Loop enforces constraints at the **runtime level** through Claude Code Hooks:

- **bound-guard.sh** — Parses DANGER ZONES from CLAUDE.md, **physically blocks** (exit 2) any edit to protected files. The agent receives a denial reason and cannot proceed.
- **root-cause-tracker.sh** — Tracks per-file edit frequency, warns when the same file is edited 3+ times (stuck loop detection).
- **drift-detector.sh** — Warns when edits span 5+ directories (scope drift detection).
- **recall-gate.sh** — Re-injects BOUND into context before compression, preventing constraint amnesia.

This is the critical difference: **BOUND is enforced by the runtime, not by the agent's good behavior.** The hooks use exit code 2 to hard-block operations, making violations physically impossible rather than merely inadvisable.
