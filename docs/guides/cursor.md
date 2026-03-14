# Cursor Integration

Ouro Loop works with Cursor IDE through `.cursorrules` files that reference the Ouro Loop methodology modules. While Cursor does not support runtime hooks like Claude Code, it can follow the full BOUND → MAP → PLAN → BUILD → VERIFY → LOOP methodology through instruction-based integration.

---

## Overview

Cursor reads `.cursorrules` at the project root to guide its AI behavior. By referencing Ouro Loop's `program.md` and your project's CLAUDE.md within `.cursorrules`, you give Cursor the same structured methodology that Claude Code uses.

!!! info "Instruction-based vs Runtime-enforced"
    Unlike Claude Code hooks that physically block DANGER ZONE edits (exit 2), Cursor integration is instruction-based. The agent is told to follow BOUND constraints but is not mechanically prevented from violating them. For critical projects, consider using Claude Code with hooks for the autonomous sessions and Cursor for interactive development.

---

## Setup

### Step 1: Create .cursorrules

Create a `.cursorrules` file in your project root that references Ouro Loop:

```markdown
# Ouro Loop Integration

## Methodology

Follow the Ouro Loop methodology defined in `~/.ouro-loop/program.md`.
Read the full file before starting any task.

## Constraints

Read and strictly follow the BOUND section in this project's `CLAUDE.md`.
The BOUND defines:
- DANGER ZONES: files you must never edit without explicit human approval
- NEVER DO: absolute prohibitions that no task justifies breaking
- IRON LAWS: measurable invariants that must always be true

## Workflow

For every task:
1. BOUND — Re-read CLAUDE.md constraints
2. MAP — Identify dependencies, failure modes, tightest constraints
3. PLAN — Decompose into severity-ordered phases
4. BUILD — Implement with RED-GREEN-REFACTOR-COMMIT
5. VERIFY — Run verification: `python ~/.ouro-loop/framework.py verify .`
6. LOOP — If verify fails, remediate autonomously. If it passes, advance.

## Verification

After each significant change, run:
```
python ~/.ouro-loop/framework.py verify .
```

Check BOUND compliance:
```
python ~/.ouro-loop/framework.py bound-check .
```

## Remediation

When verification fails:
- Read `~/.ouro-loop/modules/remediation.md` for the decision playbook
- Revert, retry with a different approach, or escalate if DANGER ZONE
- Report what you did, not what you're thinking of doing
```

### Step 2: Ensure CLAUDE.md Exists

Your project needs a `CLAUDE.md` with a BOUND section. If you haven't created one yet:

```bash
python ~/.ouro-loop/prepare.py template claude .
```

Edit the generated file to define your project's actual constraints.

### Step 3: Start a Session

Open Cursor in your project directory and describe your task. Cursor will read `.cursorrules` and follow the Ouro Loop methodology.

---

## Referencing Modules

You can reference specific Ouro Loop modules in your `.cursorrules` for deeper methodology guidance:

| Module | Path | Purpose |
|--------|------|---------|
| `bound.md` | `~/.ouro-loop/modules/bound.md` | How to define and enforce DANGER ZONES, NEVER DO, IRON LAWS |
| `map.md` | `~/.ouro-loop/modules/map.md` | Problem space analysis template |
| `plan.md` | `~/.ouro-loop/modules/plan.md` | Severity-ordered phase decomposition |
| `build.md` | `~/.ouro-loop/modules/build.md` | RED-GREEN-REFACTOR-COMMIT cycle |
| `verify.md` | `~/.ouro-loop/modules/verify.md` | Multi-layer verification gates |
| `remediation.md` | `~/.ouro-loop/modules/remediation.md` | Autonomous fix decision playbook |

Example `.cursorrules` reference:

```markdown
For complex tasks, read these modules in order:
1. `~/.ouro-loop/modules/bound.md` — constraint definition
2. `~/.ouro-loop/modules/map.md` — problem understanding
3. `~/.ouro-loop/modules/plan.md` — phase planning
```

---

## Tips for Cursor Users

1. **Use Composer for autonomous sessions** — Cursor's Composer mode is closest to Claude Code's autonomous behavior. Regular chat mode is more interactive.

2. **Run verification manually** — Since Cursor doesn't have runtime hooks, make it a habit to ask Cursor to run `framework.py verify .` after each significant change.

3. **Reference CLAUDE.md explicitly** — If Cursor seems to be drifting from BOUND constraints, paste the relevant section directly into the chat.

4. **Combine with Claude Code** — Use Claude Code (with hooks) for long autonomous sessions and Cursor for interactive development on the same project. Both read the same CLAUDE.md.

---

## Limitations

- No runtime enforcement — DANGER ZONE edits are not blocked, only discouraged
- No automatic context preservation — no recall-gate hook to re-inject BOUND on context compression
- No automatic edit tracking — no root-cause-tracker for stuck loop detection
- Agent must voluntarily run `framework.py verify` — no automatic verification triggers

For projects where runtime enforcement is critical, use [Claude Code](claude-code.md) with hooks installed.

---

## Next Steps

- [Claude Code Integration](claude-code.md) — For runtime-enforced constraints
- [Quick Start Guide](quick-start.md) — Full setup walkthrough
- [Core Concepts](../concepts.md) — Understand bounded autonomy
