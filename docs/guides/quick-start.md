# Quick Start Guide

Ouro Loop requires Python 3.10+, Git, and any AI coding agent. It has zero external dependencies — everything runs on the Python standard library. This guide walks through the complete setup process from clone to autonomous agent launch.

---

## Step 1: Clone Ouro Loop

```bash
git clone https://github.com/VictorVVedtion/ouro-loop.git ~/.ouro-loop
```

This places the framework in your home directory. You can clone it anywhere, but `~/.ouro-loop` is the conventional location.

---

## Step 2: Scan Your Project

Navigate to the project you want to apply Ouro Loop to and run the scanner:

```bash
cd /path/to/your/project
python ~/.ouro-loop/prepare.py scan .
```

The scanner analyzes your project structure and shows what Ouro Loop sees:

```
============================================================
  Ouro Loop — Project Scan
============================================================
  Project:    my-payment-service
  Types:      Python
  Files:      42       Lines:     3,200

  Languages:
    Python                  35 files  ###############
    SQL                      7 files  #######

  CLAUDE.md:  Not found
  BOUND:      Not defined
  Tests:      Found
  CI:         Found

  Recommendations:
    1. Define BOUND (DANGER ZONES, NEVER DO, IRON LAWS) before building
    2. Create CLAUDE.md with BOUND section
============================================================
```

!!! tip "What to look for"
    The scan output tells you whether your project already has a CLAUDE.md, whether BOUND constraints are defined, and whether tests and CI are present. These inform how much setup you need.

---

## Step 3: Initialize the State Directory

```bash
python ~/.ouro-loop/prepare.py init .
```

This creates the `.ouro/` directory in your project, which tracks loop state, phase progress, and verification results.

---

## Step 4: Generate a CLAUDE.md Template

```bash
python ~/.ouro-loop/prepare.py template claude .
```

This generates a `CLAUDE.md` file with placeholder sections for your BOUND definition. Open it and fill in the three BOUND layers:

### Defining Your BOUND

The BOUND is the most important part of Ouro Loop. It defines what the AI agent absolutely cannot do.

```markdown
## BOUND

### DANGER ZONES
- `src/payments/calculator.py` — financial calculations, penny-level precision
- `migrations/` — database schema, irreversible in production

### NEVER DO
- Never use float for monetary values — always Decimal
- Never delete or rename migration files
- Never commit without running the test suite

### IRON LAWS
- All monetary values use Decimal with 2-digit precision
- All API responses include request_id field
- Test coverage for payment module never drops below 90%
```

!!! warning "BOUND before code"
    Always define your BOUND before starting the autonomous loop. The BOUND is the contract between you and the agent. Without it, the agent has no constraints — and unconstrained agents are dangerous.

**How to identify DANGER ZONES:**

- Which files would cause catastrophic failure if incorrectly modified? (payments, auth, consensus)
- Which files are "load-bearing pillars" that many other modules depend on?
- Which directories contain irreversible changes? (migrations, deployment configs)

**How to write NEVER DO rules:**

- What are the invariant rules that no task justifies breaking?
- What mistakes have caused real incidents in the past?
- What coding patterns are banned for safety or compliance reasons?

**How to define IRON LAWS:**

- What measurable conditions must always be true after any change?
- What metrics serve as smoke tests for system health?
- What coverage, performance, or correctness thresholds exist?

---

## Step 5: Install Hooks (Optional but Recommended)

For Claude Code users, install the runtime enforcement hooks:

```bash
cp ~/.ouro-loop/hooks/settings.json.template .claude/settings.json
```

Edit `.claude/settings.json` to point the hook paths to your `~/.ouro-loop/hooks/` directory.

See the [Claude Code Integration Guide](claude-code.md) for detailed hook setup.

---

## Step 6: Launch the Agent

Start your AI coding agent (Claude Code, Cursor, Aider, etc.) in the project directory and prompt:

```
Read program.md from ~/.ouro-loop/ and the CLAUDE.md in this project.
Let's start the Ouro Loop for this task: [describe your task].
```

The agent will:

1. **BOUND** — Read your CLAUDE.md constraints
2. **MAP** — Understand the problem space, dependencies, failure modes
3. **PLAN** — Decompose the task into severity-ordered phases
4. **BUILD** — Implement using RED-GREEN-REFACTOR-COMMIT cycles
5. **VERIFY** — Run multi-layer verification gates
6. **LOOP** — Feed lessons back into BOUND

!!! info "What happens when verification fails"
    The agent does **not** ask for help. It consults its remediation playbook, decides on a fix (revert, retry with different approach, or escalate), executes the fix, and reports what it did. This cycle continues until verification passes or a DANGER ZONE is breached.

---

## Step 7: Monitor Progress

Use the framework CLI to check progress at any time:

```bash
python ~/.ouro-loop/framework.py status .       # Where are we in the loop?
python ~/.ouro-loop/framework.py verify .        # Run verification gates
python ~/.ouro-loop/framework.py bound-check .   # Are constraints intact?
```

Results are logged to `ouro-results.tsv`:

```
phase   verdict   bound_violations   notes
1/3     PASS      0                  transactions endpoint + tests
2/3     RETRY     0                  ROOT_CAUSE warning, fixing
2/3     PASS      0                  fixed after retry
3/3     PASS      0                  validation complete
```

---

## What's Next

- [Core Concepts](../concepts.md) — Deep dive into bounded autonomy, the BOUND system, and verification gates
- [Claude Code Integration](claude-code.md) — Install runtime enforcement hooks
- [Examples](../examples/index.md) — Real-world BOUND definitions and session logs
- [Comparison](../comparison.md) — How Ouro Loop compares to other approaches
