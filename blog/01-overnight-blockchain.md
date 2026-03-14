---
title: "I Let Claude Code Run Overnight on a Blockchain. Here's What Happened."
published: true
description: "What happens when you let an AI coding agent loose on a PBFT blockchain's consensus engine... overnight? A war story about bounded autonomy, 5 hypotheses, 4 autonomous remediations, and a root cause nobody expected."
tags: ai, claude, blockchain, autonomous-coding
canonical_url:
cover_image:
---

What happens when you let an AI coding agent loose on a PBFT blockchain's consensus engine... overnight?

Not a toy project. A real Layer 1 chain with 4 validators, live consensus, and an iron-clad rule: **SysErr rate must be 0.00%.** One wrong move in the consensus code and you fork the network. One float where there should be an integer and you've got a billion-dollar bug.

I did it anyway. Here's what happened.

## The Problem Nobody Talks About

There's a dirty secret in the "vibe coding" era: unbound AI agents break things.

Not always. Not on your TODO app or your blog template. But point an AI agent at a financial system, a consensus engine, or anything where "move fast and break things" means "lose real money," and you'll learn the lesson fast. The agent will hallucinate file paths. It'll regress architectural decisions you spent months getting right. It'll confidently "fix" a function by rewriting it in a way that passes the immediate test but violates an invariant three layers up.

The usual solution? Babysitting. You sit there, approving every file edit, reviewing every commit, answering every "should I proceed?" question. Congratulations — you now have the world's most expensive junior developer who never sleeps but constantly needs supervision.

I wanted something different. I wanted to go to sleep and wake up to results.

## The Setup: Drawing the Circle

The project was a custom L1 blockchain — Go, PBFT-inspired consensus, 4 validators. The problem: `precommit` phase latency was spiking from 4ms at idle to 100-200ms under transaction load. That kind of spike in consensus means your block times blow up, your throughput tanks, and your chain starts looking unreliable.

Before I let the agent touch anything, I drew the boundary using [Ouro Loop](https://github.com/VictorVVedtion/ouro-loop), a framework I've been building for exactly this scenario. The key concept is the **BOUND** — you define what the agent absolutely cannot do, and by doing so, you free it to do everything else.

Here's what my BOUND looked like:

```
DANGER ZONES: consensus/, p2p/gossipsub.go, state/merkle.go
NEVER DO: Never use float in consensus-critical code
IRON LAWS:
  - SysErr rate must be 0.00%
  - State root deterministic
  - Block time must not regress vs baseline under equivalent load
```

The agent could read anything, run any test, modify any non-danger-zone file, and make its own decisions about what to try next. But if it tried to edit consensus code, a runtime hook (`bound-guard.sh`) would physically block the edit with exit code 2. No amount of prompt injection or confused reasoning could override it.

The constraints aren't enforced by trusting the agent to behave. They're enforced by the runtime. That distinction matters.

I pointed the agent at the codebase, told it to investigate the `precommit` regression, and went to sleep.

## The Session: 5 Hypotheses, 4 Failures, 1 Discovery

What happened next is the part I didn't expect. I'd anticipated the agent would either solve it quickly or get stuck in a loop. Instead, it did something that looked remarkably like the systematic debugging a senior engineer would do — except it did it faster, and it caught its own mistakes.

### Hypothesis 1: CommitWait Is Too High

The agent's first instinct was reasonable: the `CommitWait` parameter was set to 80ms, which adds unnecessary delay between consensus rounds. It cut it to 40ms.

**Result:** Idle block time dropped from 111ms to 51ms. Nice. But under load? Precommit was still 100ms+. The improvement was real but it wasn't the root cause.

This is where the methodology earned its keep. Ouro Loop's ROOT_CAUSE verification gate fired: "Symptom improved, root cause not addressed." The agent kept the improvement (it was genuinely valid) but didn't declare victory. It moved on.

### Hypothesis 2: Global Mutex Contention

Next up: lock contention. The agent found a global mutex on a hot path, split it into a read-write lock, and separated the contention points.

**Result:** 6.7% improvement in lock metrics. Precommit still 90-180ms under load. Another valid micro-optimization, another ROOT_CAUSE gate failure. The agent kept the optimization and continued.

### Hypothesis 3: Goroutine Scheduling

Getting deeper: maybe the Go runtime scheduler was starving consensus goroutines under load. The agent tuned `GOMAXPROCS` and scheduler parameters.

**Result:** Zero measurable improvement. Complete dead end.

Now something interesting happened. This was the third consecutive failure, and the remediation playbook has a rule for that: **after 3 failed hypotheses, stop fixing symptoms. Step back and re-examine the architecture.**

The agent's remediation log read:

```
[REMEDIATED] gate=ROOT_CAUSE action=step_back_and_remap
  was: 3 hypotheses tested, none found root cause
  did: stopped fixing symptoms, re-examined the system architecture
  now: looking at the problem from the network layer, not consensus layer
```

That step-back — the shift from "the consensus code is slow" to "maybe the problem isn't in the consensus code at all" — was the turning point.

### Hypothesis 4: GossipSub Protocol Degradation

The agent examined the pub/sub message patterns under load. GossipSub did have overhead, but it was proportional to load, not spiking. Another dead end, but now it was looking in the right neighborhood.

### Hypothesis 5: The Real Root Cause

And then it found it.

All stress test HTTP traffic was being routed to a **single validator node**. That node was getting hammered with HTTP requests while simultaneously trying to participate in PBFT consensus. When it fell behind on `precommit` because its goroutines were busy handling HTTP, the other three validators had to **wait for it** — that's how PBFT works. One slow node slows the entire network.

The root cause wasn't in the consensus code at all. It was an infrastructure topology problem. The fix? A Caddy reverse proxy with round-robin load balancing across all 4 validators. Not a code change — an architecture change.

ROOT_CAUSE gate: **PASS.**

## The Self-Correction That Impressed Me

After deploying the load balancer fix, the agent did something I didn't expect: it tried to verify whether direct round-robin (bypassing Caddy) could recover the TPS overhead from the proxy layer. It spun up 4 separate stress test instances, each targeting one validator directly.

The results looked terrible — validator-0 fell behind again, precommit spiked to 200-500ms.

But instead of concluding "direct is worse than Caddy," the agent caught its own mistake:

```
[REMEDIATED] gate=ROOT_CAUSE action=abort_bad_experiment
  was: running 4 full stress instances (4x total load, not 1x distributed)
  did: identified that 4x stress ≠ same load distributed; killed experiment
  now: confirmed Caddy LB is the correct approach
```

Four full stress instances is **4x the total load**, not the same load distributed. The agent recognized its own flawed experiment design before it could draw a wrong conclusion from bad data. That's the ROOT_CAUSE gate preventing a false negative from corrupting the research.

I've seen senior engineers make that exact mistake and not catch it until code review.

## The Numbers

After a 30-minute soak test with all optimizations combined (CW=40ms, load balancer, lock improvements), here's the before-and-after:

| Metric | Before | After | Delta |
|--------|--------|-------|-------|
| Precommit (under load) | 100-200ms | 4ms | **-98%** |
| TPS Variance | 40.6% | 1.6% | **-96%** |
| Block time (under load) | 111-200ms | 52-57ms | **-53%** |
| Blocks/sec (soak load) | ~8.0 | ~18.5 | **+131%** |
| SysErr rate | 0.00% | 0.00% | **= (IRON LAW held)** |

That last row is the one that matters most. Throughout the entire session — 5 hypotheses, 4 remediations, multiple code changes inside a DANGER ZONE — the system error rate never budged from zero. The IRON LAW held.

And the TPS variance number tells the real production story. A blockchain that swings between 6K and 14K TPS (40.6% variance) is unreliable. One that holds steady at 10K with 1.6% variance is something you can build on.

## What This Actually Proves

This isn't a story about AI being magic. The agent didn't pull the answer out of thin air. It tested 5 hypotheses, got 4 of them wrong, and needed a forced step-back to change its frame of reference. That's just... debugging.

But here's what bounded autonomy gave us that unbounded agents or human babysitting wouldn't:

**1. The agent stayed safe while being wrong.** Four consecutive failures in consensus-adjacent code, and nothing broke. SysErr stayed at 0.00%. The BOUND held. Being wrong didn't mean being dangerous.

**2. The agent kept partial wins.** CommitWait reduction and lock optimization were genuine improvements even though they weren't the root cause. The methodology correctly separated "valid improvement" from "root cause found." An unbounded agent might have reverted everything after finding the real fix.

**3. The step-back was systematic, not lucky.** After 3 failures, the remediation playbook forced an architectural re-examination. That's not the agent being brilliant — it's the methodology preventing tunnel vision. The agent was told: "You've been looking at consensus code for 3 rounds and you're still stuck. Look somewhere else." It worked.

**4. The agent caught its own bad science.** The flawed experiment with 4x stress load could have led to a wrong conclusion that persisted. The verification gate caught it. Bounded autonomy isn't just about preventing bad code — it's about preventing bad reasoning.

**5. The real cause was invisible to code-level thinking.** An HTTP routing problem doesn't show up in code review. It doesn't show up in unit tests. It shows up in system behavior under load. The agent's ability to run stress tests, analyze results, and reason about infrastructure topology — while being constrained from breaking anything — is the sweet spot.

## Try It Yourself

[Ouro Loop](https://github.com/VictorVVedtion/ouro-loop) is open source and has zero dependencies. Three files that matter: `program.md` (the methodology), `framework.py` (the runtime), and your project's `CLAUDE.md` (the boundary).

The full session log from this blockchain investigation — every hypothesis, every remediation, every verification gate — is in the repo under `examples/blockchain-l1/session-log.md`.

The framework works with Claude Code, Cursor, Aider, Codex, or any AI agent that can read Markdown and run terminal commands. Define your BOUND, point your agent, and let it run.

The constraint space defines the creative space. Draw the circle. State the laws. Release the loop.
