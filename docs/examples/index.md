# Examples

Ouro Loop includes real-world BOUND definitions and session logs from four distinct project domains. Each example demonstrates how bounded autonomy adapts to different constraint profiles — from blockchain consensus safety to financial precision to ML experiment discipline.

---

## Example Domains

### :material-cube-outline: Blockchain L1

A custom Layer 1 blockchain (PBFT consensus, 4 validators) where the agent investigated a consensus performance regression. DANGER ZONES protect consensus, cryptography, and state root integrity.

**Highlights:**

- 5 hypotheses tested, 4 autonomous remediations
- ROOT_CAUSE gate fired 4 times, preventing premature conclusions
- Real root cause was architectural (HTTP routing), not code-level
- SysErr rate maintained at 0.00% throughout (IRON LAW)

[:material-arrow-right: View Example](blockchain-l1.md) | [:material-file-document: Full Session Log](../session-logs/blockchain-l1.md)

---

### :material-cellphone: Consumer Product

A creative collaboration iOS/macOS app where the agent remediated ESLint errors in a React/Next.js frontend. DANGER ZONES protect the audio engine, CRDT conflict resolver, and IAP module.

**Highlights:**

- ROOT_CAUSE gate caught a lazy fix (restructuring an effect vs. eliminating it)
- Agent pushed toward architecturally superior derived-state pattern
- Simple complexity correctly identified — no phase plan needed
- Build pass IRON LAW served as safety net

[:material-arrow-right: View Example](consumer-product.md) | [:material-file-document: Full Session Log](../session-logs/consumer-product.md)

---

### :material-cash: Financial System

A real-money gaming platform with wallet management, bet settlement, and withdrawal processing. DANGER ZONES protect balance calculations, settlement state machine, and migration files.

**Highlights:**

- Decimal precision IRON LAWS (never float for money)
- Atomic balance changes (debit + credit in single transaction)
- 95% minimum test coverage for financial modules
- Immutable audit trail for all settlement state transitions

[:material-arrow-right: View Example](financial-system.md)

---

### :material-brain: ML Research

An autoresearch-style autonomous ML experiment framework where the agent iterates on `train.py` to minimize `val_bpb`. DANGER ZONES protect the evaluation harness and data pipeline.

**Highlights:**

- Single-metric optimization (val_bpb) reframed as BOUND
- 5-minute training budget as an IRON LAW
- Only `train.py` is modifiable — everything else is fixed
- Regressions auto-revert, improvements keep the commit

[:material-arrow-right: View Example](ml-research.md)

---

## Patterns Across Examples

Despite the different domains, several patterns emerge:

1. **DANGER ZONES protect the irreversible** — consensus logic, financial calculations, data migrations, evaluation harnesses. These are the files where a mistake is catastrophically expensive.

2. **IRON LAWS are measurable** — SysErr rate, Decimal precision, test coverage percentage, training time budget. Every IRON LAW can be verified programmatically.

3. **NEVER DO rules encode hard-won lessons** — "Never use float for money" and "Never run benchmarks against a single node" are rules born from real incidents.

4. **BOUND grows after each session** — The LOOP stage feeds new DANGER ZONES, NEVER DO rules, and IRON LAWS back into CLAUDE.md, making each subsequent session safer.

---

## Contributing Examples

The most valuable contributions to Ouro Loop are real-world BOUND definitions from complex domains. If you've used Ouro Loop to bound an agent in a domain not covered here, submit a sanitized CLAUDE.md and session log to `examples/`. See [CONTRIBUTING.md](https://github.com/VictorVVedtion/ouro-loop/blob/main/CONTRIBUTING.md).
