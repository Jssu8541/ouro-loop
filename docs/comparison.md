# Comparison

Ouro Loop occupies a specific position on the autonomy spectrum for AI coding agents: it provides bounded autonomy with runtime enforcement, sitting between human-in-the-loop tools (Cursor, Copilot) that require constant developer attention and unbounded agents (raw LLM prompting) that lack safety constraints. This page compares Ouro Loop against static rule files, monitoring-only tools, and raw LLM agents across key dimensions.

---

## The Autonomy Spectrum

```
Complete Manual  ← ──────────────────────────────── →  Complete Autonomy
   IDE only       human-in-loop     bounded autonomy      unbounded agent
                 (Cursor/Copilot)    (Ouro Loop)           (raw LLM)
```

| Position | Description | Failure Mode |
|----------|-------------|-------------|
| **IDE only** | Human writes all code, AI provides completions | Slowest. No autonomy. |
| **Human-in-the-loop** | Agent proposes, human approves. Constant back-and-forth. | Developer becomes a babysitter. Cannot run overnight. |
| **Bounded autonomy** | Agent operates freely within enforced constraints. Detects, decides, acts, reports. | Requires upfront BOUND definition. Setup overhead for simple tasks. |
| **Unbounded agent** | Agent has full freedom. No constraints, no verification. | Hallucination, drift, constraint violation, context decay. |

Ouro Loop targets the **bounded autonomy** zone: maximum agent freedom with minimum catastrophic risk.

---

## Ouro Loop vs Static Rule Files

Static rule files (`.cursorrules`, `CLAUDE.md` instructions, `.aider.conf`) tell the agent what to do but cannot enforce compliance.

| Dimension | Static Rules | Ouro Loop |
|-----------|-------------|-----------|
| **Constraint definition** | Markdown instructions in a config file | BOUND system: DANGER ZONES + NEVER DO + IRON LAWS |
| **Enforcement mechanism** | Agent's good behavior (can be ignored) | Runtime hooks with exit 2 hard-block (physically impossible to bypass) |
| **Verification** | None — agent self-reports | Multi-layer gates: EXIST, RELEVANCE, ROOT_CAUSE, RECALL, MOMENTUM |
| **On failure** | Agent asks human or silently proceeds | Autonomous remediation: detect, decide, act, report |
| **State tracking** | None | Phase progress, verification history, remediation log |
| **Drift detection** | None | Drift detector hook warns at 5+ directory scope |
| **Context decay** | Agent forgets constraints over time | Recall gate re-injects BOUND before context compression |
| **Overnight operation** | Risky — no safety net | Designed for it — continuous verification loop |

**Bottom line:** Static rules are suggestions. Ouro Loop constraints are enforced by the runtime.

---

## Ouro Loop vs Monitoring Tools

Monitoring and observability tools (linters, CI checks, static analysis, code review bots) detect problems but rely on humans to fix them.

| Dimension | Monitoring Tools | Ouro Loop |
|-----------|-----------------|-----------|
| **Detection** | Automated (linting, tests, coverage) | Automated (5 verification gates + custom checks) |
| **Response to failure** | Alert human, create issue, block merge | Agent autonomously remediates: revert, retry, try alternative approach |
| **Action model** | Detect → Alert → Wait for human | Detect → Decide → Act → Report |
| **Loop closure** | Human fixes, re-runs check | Agent fixes, re-verifies, advances or retries |
| **Learning** | Static rules updated manually | LOOP stage feeds discoveries back into BOUND after each session |
| **Scope** | Per-check (lint, test, coverage) | Per-session (multi-phase, multi-check, continuous) |
| **Autonomous operation** | No — requires human response | Yes — within BOUND |

**Bottom line:** Monitoring tools detect. Ouro Loop detects, decides, and acts.

---

## Ouro Loop vs Raw LLM Agents

Raw LLM agents (ChatGPT with code execution, Claude without constraints, unstructured agent prompts) have full autonomy with no guardrails.

| Dimension | Raw LLM Agent | Ouro Loop |
|-----------|--------------|-----------|
| **Constraint awareness** | None unless prompted | BOUND formally defined and runtime-enforced |
| **Hallucination prevention** | Hope-based | EXIST gate checks file/API existence before proceeding |
| **Drift prevention** | None | RELEVANCE gate + drift detector hook |
| **Stuck loop prevention** | None | ROOT_CAUSE gate + root-cause tracker hook (warns at 3+ edits) |
| **Context decay** | Agent silently forgets | RECALL gate + recall-gate hook re-injects BOUND |
| **Self-correction** | Random retry or give up | Structured remediation playbook with escalation rules |
| **Auditability** | Chat history only | Verification logs, phase results, remediation reports |
| **Reproducibility** | None | Structured phases, logged decisions, TSV result history |

**Bottom line:** Raw agents are powerful but uncontrolled. Ouro Loop channels that power within safe boundaries.

---

## Ouro Loop vs autoresearch

Ouro Loop was directly inspired by [karpathy/autoresearch](https://github.com/karpathy/autoresearch). Both share the same core idea: give an AI agent a loop and let it iterate autonomously.

| Dimension | autoresearch | Ouro Loop |
|-----------|-------------|-----------|
| **Domain** | ML training experiments | General software engineering |
| **Human programs** | `program.md` (experiment strategy) | `program.md` (dev strategy) + `CLAUDE.md` (boundaries) |
| **AI modifies** | `train.py` (model code) | Target project code + `framework.py` |
| **Fixed constraint** | 5-minute training budget | BOUND (DANGER ZONES, NEVER DO, IRON LAWS) |
| **Core metric** | val_bpb (lower is better) | Multi-layer verification (gates + self-assessment) |
| **On failure** | Auto-revert, try next experiment | Auto-remediate, try alternative approach |
| **Read-only files** | `prepare.py` | `prepare.py` + `modules/` |
| **Scope** | Single-file optimization | Multi-file, multi-phase development |

**Bottom line:** autoresearch pioneered the autonomous loop for ML. Ouro Loop generalizes it to all software engineering with formal constraint enforcement.

---

## When to Use What

| Scenario | Recommended Approach |
|----------|---------------------|
| Quick prototype, hackathon | Static rules or no framework |
| Interactive pair programming | Human-in-the-loop (Cursor, Copilot) |
| Overnight autonomous builds | **Ouro Loop** |
| Long-running refactors | **Ouro Loop** |
| Production code with critical constraints | **Ouro Loop** |
| ML experiment iteration | autoresearch or Ouro Loop with ML BOUND |
| Simple scripts, single-file projects | Raw LLM agent is sufficient |
