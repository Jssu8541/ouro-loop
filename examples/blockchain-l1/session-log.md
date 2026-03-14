# Session Log: Consensus Performance Regression Under Load

A real Ouro Loop session on a Layer 1 blockchain project. Names and specifics anonymized.

## Context

- **Project type**: Custom L1 blockchain (Go), PBFT-inspired consensus, 4 validators
- **Task**: Investigate why `precommit` phase latency spikes from 4ms (idle) to 100-200ms under transaction load
- **BOUND interaction**: Inside DANGER ZONE (`consensus/`)

## BOUND (from CLAUDE.md)

```
DANGER ZONES: consensus/, p2p/gossipsub.go, state/merkle.go
NEVER DO: Never use float in consensus-critical code
IRON LAWS:
  - SysErr rate must be 0.00%
  - State root deterministic
  - Block time must not regress vs baseline under equivalent load
```

## MAP

```
User expects:    block time stays near idle performance under load
Failure modes:   consensus stall, round escalation storm, chain halt
Tightest constraint: SysErr 0.00% (IRON LAW)
Dependencies:    consensus → p2p gossip → HTTP gateway → stress test harness
Reusable:        existing stress test binary, Tier 10 baseline data
Success metric:  precommit < 10ms under load, block time < 100ms
```

## PLAN

Complexity: **Complex** (DANGER ZONE, multiple subsystems, unknown root cause)
Approach: Hypothesis-driven elimination, not phase-based construction.

## BUILD + VERIFY + REMEDIATE (The Root Cause Hunt)

### Hypothesis 1: CommitWait parameter too high

```
Action:   Reduced CommitWait from 80ms to 40ms
Result:   Idle block time halved (111ms → 51ms)
          Load block time still 122-234ms — precommit still 100ms+
Verdict:  PARTIAL — improved idle, didn't fix load regression
```

VERIFY: ROOT_CAUSE gate → FAIL. Symptom improved, root cause not addressed.

```
[REMEDIATED] gate=ROOT_CAUSE action=keep_partial_and_continue
  was: CW=40ms improved idle but load still regressed
  did: kept CW=40ms (valid improvement), continued investigation
  now: investigating lock contention
  bound: inside DANGER ZONE — proceeding with caution
```

### Hypothesis 2: Global mutex contention

```
Action:   Split globalMu into read-write lock, separated hot paths
Result:   6.7% improvement in lock contention metrics
          precommit still 90-180ms under load
Verdict:  FAIL — marginal improvement, not the root cause
```

VERIFY: ROOT_CAUSE gate → FAIL.

```
[REMEDIATED] gate=ROOT_CAUSE action=discard_hypothesis
  was: lock optimization only 6.7% improvement
  did: kept the optimization (valid), moved to next hypothesis
  now: investigating goroutine scheduling
  bound: DANGER ZONE changes minimal and tested
```

### Hypothesis 3: Goroutine scheduling starvation

```
Action:   Set GOMAXPROCS=8, tuned scheduler parameters
Result:   No measurable improvement
Verdict:  FAIL — not the cause
```

VERIFY: ROOT_CAUSE gate → FAIL. Third consecutive failure.

```
[REMEDIATED] gate=ROOT_CAUSE action=step_back_and_remap
  was: 3 hypotheses tested, none found root cause
  did: stopped fixing symptoms, re-examined the system architecture
  now: looking at the problem from the network layer, not consensus layer
  bound: no DANGER ZONE code changed in this step
```

### Hypothesis 4: GossipSub protocol degradation

```
Action:   Analyzed GossipSub v1/v2/v3 message patterns under load
Result:   Protocol overhead exists but is proportional, not the spike cause
Verdict:  FAIL — GossipSub is a contributor, not the root cause
```

### Hypothesis 5: Single-point HTTP gateway bottleneck

```
Action:   Examined network topology
Finding:  ALL stress test HTTP traffic routed to a single validator node
          That node's precommit delays because it's saturated with HTTP
          Other 3 validators wait for it in PBFT consensus
Root cause: Single-node HTTP bottleneck creates consensus-wide delay
```

VERIFY: ROOT_CAUSE gate → **PASS**.

```
Solution: Caddy reverse proxy with round-robin load balancing across all 4 validators
Result:
  - precommit: 100-200ms → 4ms (back to idle levels)
  - block time under load: 122-234ms → 92ms
  - SysErr: 0.00% (IRON LAW maintained throughout)
```

## Results

```
  phase:        investigation (5 hypotheses)
  verdict:      PASS
  bound_check:  PASS — SysErr 0.00%, no IRON LAW violated
  metric:       precommit 4ms (target: <10ms)
  block_time:   92ms under load (baseline idle: 51ms)
  remediation:  4 autonomous remediations before finding root cause
```

## LOOP — What fed back into BOUND

New additions to CLAUDE.md after this session:

```
DANGER ZONES (added):
  - infrastructure/caddy/ — load balancer config, single point of failure if misconfigured

IRON LAWS (added):
  - Stress tests must distribute load across all validators, never single-node
  - Load balancer health checks must be configured before any performance test

NEVER DO (added):
  - Never run performance benchmarks against a single validator endpoint
```

## Methodology Observations

1. **ROOT_CAUSE gate fired 4 times** — each time it correctly identified that we were fixing symptoms, not the cause. The gate prevented premature celebration after partial improvements.

2. **The 3-failure step-back rule worked** — after 3 consecutive failed hypotheses, the remediation playbook told us to "stop fixing, re-examine architecture." This led to looking at the network layer instead of the consensus layer, which found the actual root cause.

3. **Partial improvements were kept** — CW=40ms and the lock optimization were genuine improvements even though they weren't the root cause. The methodology correctly distinguished between "valid improvement" and "root cause found."

4. **BOUND was respected throughout** — despite working inside a DANGER ZONE (consensus/), SysErr stayed at 0.00% and no IRON LAW was violated. The agent operated autonomously within the boundary.

5. **The real root cause was architectural, not code-level** — it was a deployment topology issue (HTTP routing), not a code bug. The methodology's MAP stage should have caught this earlier by mapping the "Dependencies" dimension more carefully. This is a lesson for future MAP stages.
