# Social Media Launch Posts

---

## Show HN

**Title**: Show HN: Ouro Loop – Autonomous AI coding with runtime-enforced guardrails

**Text**:
Hey HN,

I built Ouro Loop because I got tired of babysitting AI coding agents. Every time I'd let Claude Code or Cursor run on a complex task, it would eventually hallucinate file paths, break production constraints, or get stuck in fix-break loops.

Ouro Loop implements "bounded autonomy" — you define absolute constraints upfront (DANGER ZONES files that can't be touched, IRON LAWS that must always be true, NEVER DO rules), and the agent loops autonomously within them: Build → Verify → Self-Fix.

Key differences from just using .cursorrules or CLAUDE.md:

1. **Runtime enforcement**: Claude Code Hooks use exit code 2 to physically block edits to DANGER ZONE files. The agent can't bypass this.

2. **Autonomous remediation**: When verification fails, the agent doesn't ask you — it reverts, tries a different approach, and reports what it did.

3. **Verification gates**: EXIST (anti-hallucination), ROOT_CAUSE (anti-stuck-loop), RELEVANCE (anti-drift), RECALL (anti-context-decay), MOMENTUM (anti-stall).

Real result: On a PBFT blockchain, the agent tested 5 hypotheses, self-remediated 4 failures, and found a root cause that was architectural (HTTP routing), not code-level. Precommit latency went from 200ms → 4ms. TPS variance from 40.6% → 1.6%. SysErr stayed at 0.00% throughout.

Inspired by Karpathy's autoresearch (autonomous ML experimentation), extended to general software engineering. Zero dependencies, pure Python 3.10+, 3 files that matter.

https://github.com/VictorVVedtion/ouro-loop

---

## Reddit r/ClaudeAI

**Title**: I built an open-source framework that lets Claude Code run autonomously overnight with runtime-enforced guardrails

**Text**:
After too many sessions of babysitting Claude Code on complex tasks, I built Ouro Loop — a framework for "bounded autonomy."

The idea: you define absolute constraints in your CLAUDE.md (DANGER ZONES = files that must never be touched, IRON LAWS = invariants that must always be true, NEVER DO = absolute prohibitions). Then the agent loops autonomously: Build → Verify → Self-Fix.

The key innovation is **runtime enforcement**: 4 Claude Code Hooks that use exit code 2 to physically block the agent from editing protected files. It's not "please don't touch payments.py" — it's "you literally cannot touch payments.py without my approval."

When verification fails, the agent doesn't ask for help. It consults its remediation playbook, reverts if needed, tries a different approach, and reports what it did.

Real test on a blockchain: agent tested 5 hypotheses, self-remediated 4 failures, found the root cause was architectural (not code-level). Precommit: 200ms → 4ms. TPS variance: 40.6% → 1.6%.

Inspired by Karpathy's autoresearch. Zero dependencies. Works with Claude Code, Cursor, Aider, Codex.

GitHub: https://github.com/VictorVVedtion/ouro-loop

---

## Reddit r/programming

**Title**: Ouro Loop: An open-source framework for running AI coding agents autonomously with runtime-enforced constraints

**Text**:
The "vibe coding" era has a problem: AI agents hallucinate file paths, break production constraints, and get stuck in infinite fix-break loops. The current solution (human-in-the-loop) negates the promise of autonomous coding.

Ouro Loop takes a different approach: **bounded autonomy**. You define absolute constraints (DANGER ZONES, IRON LAWS, NEVER DO rules), then let the agent loop autonomously within them.

What makes it different from static rules files:
- Runtime enforcement via Claude Code Hooks (exit 2 = hard block)
- Autonomous remediation (agent fixes its own failures, doesn't ask humans)
- Multi-layer verification gates (EXIST, ROOT_CAUSE, RELEVANCE, RECALL, MOMENTUM)

Zero dependencies, pure Python 3.10+, works with any AI coding agent.

Real results from a blockchain session: precommit latency 200ms → 4ms, TPS variance 40.6% → 1.6%, with the IRON LAW (SysErr = 0.00%) maintained throughout.

https://github.com/VictorVVedtion/ouro-loop

---

## Reddit r/Python

**Title**: Ouro Loop — Zero-dependency Python framework for autonomous AI coding with guardrails

**Text**:
Built a lightweight Python framework (3.10+, zero dependencies, standard library only) for running AI coding agents autonomously with runtime-enforced constraints.

The core idea: define what the agent must NEVER do (BOUND system), then let it loop through Build → Verify → Self-Fix cycles without human intervention.

- `framework.py` — State machine, verification gates, logging CLI
- `prepare.py` — Project scanning and initialization
- `program.md` — Methodology instructions for the agent

The verification system runs 5 gates: EXIST (anti-hallucination), ROOT_CAUSE (anti-stuck-loop), RELEVANCE (anti-drift), RECALL (anti-context-decay), MOMENTUM (anti-stall).

When verification fails, the agent autonomously remediates — reverts, retries with a different approach, reports what it did. No human intervention needed (unless a DANGER ZONE is touched).

Works with Claude Code, Cursor, Aider, Codex, or any agent that can read markdown and run Python.

https://github.com/VictorVVedtion/ouro-loop

---

## X/Twitter Launch Thread

**Thread (10 tweets)**:

1/ What happens when you let an AI agent code overnight on a blockchain's consensus engine?

I built Ouro Loop — a framework for "bounded autonomy" that lets AI agents run autonomously with runtime-enforced guardrails.

Here's what happened. 🧵

2/ The problem: "vibe coding" agents hallucinate paths, break constraints, and get stuck in fix-break loops.

Current solution? Pause and ask humans.

That's not autonomy — it's micro-managing a tireless junior dev.

3/ The fix: define what the agent must NEVER do — BEFORE it writes code.

DANGER ZONES (untouchable files)
IRON LAWS (always-true invariants)
NEVER DO (absolute prohibitions)

This is the BOUND system. The agent's constitution.

4/ But instructions aren't enough. Agents ignore them.

So we enforce at the RUNTIME level:
- Claude Code Hooks use exit code 2 to PHYSICALLY block edits to DANGER ZONE files
- The agent can't bypass this. It's not a suggestion.

5/ When verification fails inside the boundary, the agent doesn't ask for help.

It consults its remediation playbook:
→ Revert
→ Try a different approach
→ Report what it did

This is autonomous remediation. The agent eats its own errors.

6/ Real test: PBFT blockchain, 4 validators.

The agent tested 5 hypotheses:
❌ CommitWait tuning (partial)
❌ Lock contention (marginal)
❌ Goroutine scheduling (nothing)
❌ GossipSub protocol (not root cause)
✅ HTTP routing bottleneck (architectural!)

7/ After 3 failures, the step-back rule kicked in:

"Stop fixing symptoms. Re-examine the architecture."

The root cause wasn't code — it was deployment topology. A single-node HTTP bottleneck was starving consensus.

8/ Results:

Precommit: 200ms → 4ms (-98%)
TPS variance: 40.6% → 1.6% (-96%)
SysErr: 0.00% throughout (IRON LAW maintained)
Blocks/sec: 8 → 18.5 (+131%)

4 autonomous remediations. Zero human intervention.

9/ The agent even caught its own flawed experiment.

When testing an alternative, it ran 4x full stress instead of 1x distributed. It identified the design flaw before drawing wrong conclusions.

The ROOT_CAUSE gate prevents false conclusions, not just false fixes.

10/ Ouro Loop:
- Zero dependencies, pure Python 3.10+
- Works with Claude Code, Cursor, Aider, Codex
- 4 Claude Code Hooks for runtime enforcement
- Real session logs included
- Inspired by @karpathy's autoresearch

Try it: github.com/VictorVVedtion/ouro-loop

Stop babysitting. Draw the circle. Release the loop.
