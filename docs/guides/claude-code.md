# Claude Code Integration

Ouro Loop provides four Claude Code Hooks that enforce BOUND constraints at the tool level. When installed, the agent **physically cannot** edit DANGER ZONE files without user approval — regardless of what instructions it receives. This is what makes Ouro Loop different from instruction-only approaches: constraints are enforced by the runtime, not by the agent's good behavior.

---

## Overview of Hooks

| Hook | Event | Purpose |
|------|-------|---------|
| `bound-guard.sh` | PreToolUse:Edit/Write | Parses CLAUDE.md DANGER ZONES and **blocks** edits to protected files (exit 2 hard-block) |
| `root-cause-tracker.sh` | PostToolUse:Edit/Write | Tracks per-file edit count, warns at 3+ edits, strongly warns at 5+ |
| `drift-detector.sh` | PreToolUse:Edit/Write | Warns when edits span 5+ directories (scope drift detection) |
| `recall-gate.sh` | PreCompact | Re-injects BOUND section into context before compression |

---

## Installation

### Step 1: Copy the Settings Template

```bash
# Ensure .claude directory exists
mkdir -p .claude

# Copy the template
cp ~/.ouro-loop/hooks/settings.json.template .claude/settings.json
```

### Step 2: Edit Hook Paths

Open `.claude/settings.json` and update the command paths to point to your Ouro Loop installation:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "command": "bash ~/.ouro-loop/hooks/bound-guard.sh"
      },
      {
        "matcher": "Edit|Write",
        "command": "bash ~/.ouro-loop/hooks/drift-detector.sh"
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "command": "bash ~/.ouro-loop/hooks/root-cause-tracker.sh"
      }
    ],
    "PreCompact": [
      {
        "command": "bash ~/.ouro-loop/hooks/recall-gate.sh"
      }
    ]
  }
}
```

### Step 3: Verify Installation

Start Claude Code in your project and try to edit a DANGER ZONE file. You should see the hook block the operation:

```
Hook blocked edit: src/payments/calculator.py is in DANGER ZONE.
BOUND says: financial calculations, penny-level precision.
Exit 2 — hard block. Request human approval to proceed.
```

---

## How Each Hook Works

### bound-guard.sh — DANGER ZONE Enforcement

**Event:** PreToolUse (before Edit or Write)

This hook parses your CLAUDE.md for the `### DANGER ZONES` section, extracts all protected file patterns, and checks whether the agent's target file matches any of them.

- **Match found:** Exits with code 2 (hard-block). The agent cannot proceed.
- **No match:** Exits with code 0 (silent pass). Zero overhead for safe files.

!!! note "Self-protection"
    The hook protects itself. If the agent tries to edit `hooks/bound-guard.sh`, it is blocked — preventing the agent from disabling its own constraints.

**Verification example:**

```bash
# This should be BLOCKED (DANGER ZONE)
# Agent tries to edit: src/payments/calculator.py
# → Hook output: "DANGER ZONE — blocked"

# This should PASS silently (safe file)
# Agent tries to edit: src/utils/helpers.py
# → No output, edit proceeds
```

### root-cause-tracker.sh — Stuck Loop Detection

**Event:** PostToolUse (after Edit or Write)

Tracks how many times each file has been edited in the current session. This detects the common failure mode where an AI agent edits the same file repeatedly, stuck in a fix-break loop.

- **3+ edits:** Warning message suggesting the agent check for root cause
- **5+ edits:** Strong warning recommending the agent step back and re-examine

The tracker stores counts in a temporary file within `.ouro/`.

### drift-detector.sh — Scope Drift Detection

**Event:** PreToolUse (before Edit or Write)

Monitors the breadth of directories being touched. When edits span 5 or more distinct directories, the hook warns that scope may be drifting from the original task.

This is a **warning** (exit 0), not a block. The agent can proceed but receives a nudge to check relevance.

### recall-gate.sh — Context Preservation

**Event:** PreCompact (before context window compression)

When Claude Code compresses the conversation context, critical information like the BOUND definition can be lost. This hook:

1. Reads the BOUND section from CLAUDE.md
2. Injects it into the compressed context
3. Ensures the agent remembers its constraints even after context compaction

This prevents **constraint amnesia** during long autonomous sessions.

---

## Troubleshooting

### Hooks Not Firing

!!! warning "Common issue"
    Make sure `.claude/settings.json` exists in your **project directory** (not in `~/.claude/`). Claude Code reads hooks from the project-level settings.

1. Verify the file exists: `cat .claude/settings.json`
2. Check that the hook paths are correct and the scripts are executable
3. Restart Claude Code after modifying `settings.json`

### Hook Blocks Everything

If the hook is blocking files that should be allowed:

1. Check your CLAUDE.md DANGER ZONES section — are the patterns too broad?
2. A pattern like `src/` would block everything under `src/`. Use specific paths.
3. Review the hook's parsing logic to ensure your DANGER ZONES format is correct

### Sound Alerts Not Working

Sound effects (used by Watchdog, not by hooks directly) require macOS. On other platforms, sounds fail silently via `2>/dev/null &`.

---

## Verified Behavior

These behaviors have been verified in live Claude Code sessions:

| Target File | Expected | Result |
|-------------|----------|--------|
| `framework.py` (DANGER ZONE) | BLOCKED | Exit 2, agent sees denial reason |
| `hooks/bound-guard.sh` (DANGER ZONE) | BLOCKED | Hook protects itself |
| `CONTRIBUTING.md` (safe file) | PASS | Silent, zero overhead |
| `src/utils/helpers.py` (safe file) | PASS | Silent, zero overhead |

---

## Next Steps

- [Quick Start Guide](quick-start.md) — Full setup walkthrough
- [Core Concepts](../concepts.md) — Understand the BOUND system in depth
- [Cursor Integration](cursor.md) — Using Ouro Loop with Cursor IDE
