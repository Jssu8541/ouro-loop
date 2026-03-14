# Blockchain L1 Example

This example demonstrates Ouro Loop applied to a custom Layer 1 blockchain with PBFT-inspired consensus. The agent investigated a consensus performance regression where `precommit` latency spiked from 4ms to 200ms under transaction load. Working inside DANGER ZONES (consensus/), the agent tested 5 hypotheses, autonomously remediated 4 failures, and discovered a root cause that was architectural — not code-level.

---

## Project Profile

| | |
|---|---|
| **Language** | Rust (no_std compatible core, async networking via tokio) |
| **Architecture** | PBFT consensus, UTXO transaction model, Merkle Patricia Trie |
| **Validators** | 4-node network |
| **Task** | Investigate precommit latency spike under load |

---

## BOUND Definition

### DANGER ZONES

```
consensus/           — Consensus engine. Incorrect changes = network fork.
crypto/              — Cryptographic primitives. Wrong implementation = funds theft.
state/merkle.rs      — Merkle tree. State root integrity depends on this.
vm/executor.rs       — VM execution. Gas calculation errors = DoS vector.
p2p/protocol.rs      — Network protocol. Breaking changes split the network.
```

### NEVER DO

- Never change serialization format without a version bump
- Never modify consensus threshold constants without formal analysis
- Never use unsafe Rust in crypto modules
- Never skip fuzzing for parser/deserializer changes
- Never merge code that changes state root calculation without 3 independent test vectors
- Never weaken signature verification, even for "performance"
- Never introduce floating point in consensus-critical code

### IRON LAWS

- All consensus-critical code is deterministic
- All cryptographic operations use constant-time implementations
- State root is computed identically by all nodes (byte-level determinism)
- Gas costs are monotonically non-decreasing with operation complexity
- Block validation is a pure function of (block, parent_state)
- All network messages are backwards-compatible for 2+ major versions
- Fuzzing corpus grows monotonically — never remove test cases

---

## The Root Cause Hunt

The agent followed a hypothesis-driven elimination approach, testing each candidate cause and using the ROOT_CAUSE verification gate to determine whether the actual root cause had been found.

### Hypothesis 1: CommitWait Parameter Too High

Reduced CommitWait from 80ms to 40ms. Idle block time halved (111ms to 51ms), but load block time still 122-234ms. ROOT_CAUSE gate fired — symptom improved, root cause not addressed. Agent kept the partial improvement and continued.

### Hypothesis 2: Global Mutex Contention

Split globalMu into read-write lock, separated hot paths. 6.7% improvement in lock contention metrics, but precommit still 90-180ms under load. ROOT_CAUSE gate fired — marginal, not the cause. Agent kept the optimization and continued.

### Hypothesis 3: Goroutine Scheduling Starvation

Set GOMAXPROCS=8, tuned scheduler. No measurable improvement. ROOT_CAUSE gate fired for the third consecutive time, triggering the **3-failure step-back rule**: "stop fixing symptoms, re-examine the architecture." This led the agent to look at the network layer instead of the consensus layer.

### Hypothesis 4: GossipSub Protocol Degradation

Analyzed GossipSub message patterns under load. Protocol overhead proportional, not the spike cause. Another dead end, but narrowing the search space.

### Hypothesis 5: Single-Point HTTP Gateway Bottleneck

Examined network topology and found the root cause: **all stress test HTTP traffic was routed to a single validator node**. That node's precommit delayed because it was saturated with HTTP requests, and the other 3 validators waited for it in PBFT consensus.

!!! success "ROOT_CAUSE gate: PASS"
    The fix was a Caddy reverse proxy with round-robin load balancing across all 4 validators. The root cause was a deployment topology issue, not a code bug.

---

## Results

| Metric | Before | After | Delta |
|--------|--------|-------|-------|
| Precommit (under load) | 100-200ms | 4ms | **-98%** |
| Block time (idle) | 111ms | 52ms | **-53%** |
| Block time (soak load) | 111-200ms | 52-57ms | **-53%** |
| Blocks/sec (soak load) | ~8.0 | ~18.5 | **+131%** |
| TPS Variance | 40.6% | 1.6% | **-96%** |
| SysErr rate | 0.00% | 0.00% | = (IRON LAW) |

---

## What Fed Back into BOUND

After the session, new constraints were added to CLAUDE.md:

```markdown
DANGER ZONES (added):
  - infrastructure/caddy/ — load balancer config, single point of failure

IRON LAWS (added):
  - Stress tests must distribute load across all validators
  - Load balancer health checks must be configured before performance tests

NEVER DO (added):
  - Never run performance benchmarks against a single validator endpoint
```

This is the LOOP stage in action: lessons from one session make the next session safer.

---

## Methodology Observations

1. **ROOT_CAUSE gate fired 4 times** — each time correctly identifying symptom-level fixes
2. **The 3-failure step-back rule worked** — redirected investigation from consensus layer to network layer
3. **Partial improvements were kept** — CommitWait and lock optimizations were valid even though they weren't the root cause
4. **The agent caught its own bad experiment** — identified that 4 separate stress instances = 4x total load, not 1x distributed
5. **Soak stability was the real breakthrough** — TPS variance dropping from 40.6% to 1.6% matters more than raw throughput

[:material-file-document: Full Session Log](../session-logs/blockchain-l1.md)
