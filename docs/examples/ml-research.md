# ML Research Example

An autoresearch-style BOUND definition for autonomous ML experimentation, reframing Karpathy's single-metric optimization as a BOUND system.

## BOUND Definition

### DANGER ZONES

- `train.py` lines 1-50 — Core training loop setup, hyperparameter definitions
- `data/` — Training and validation datasets
- `checkpoints/best_model.pt` — Best model checkpoint, never overwrite without improvement

### NEVER DO

- Never train for more than 5 minutes per experiment
- Never modify the validation dataset
- Never change the evaluation metric definition
- Never skip validation after training
- Never delete experiment logs
- Never use the test set during development

### IRON LAWS

- `val_bpb` (validation bits-per-byte) must improve or experiment is reverted
- All experiments logged with hyperparameters, metrics, and duration
- Training budget: 5 minutes maximum per experiment
- Model architecture changes require validation against baseline
- Random seeds are fixed and recorded for reproducibility

## Relationship to autoresearch

This example directly maps Karpathy's [autoresearch](https://github.com/karpathy/autoresearch) paradigm:

| autoresearch | Ouro Loop |
|---|---|
| 5-minute training budget | IRON LAW: max 5 min per experiment |
| `val_bpb` must improve | IRON LAW: val_bpb improvement required |
| Auto-revert on regression | NEVER DO: don't overwrite best checkpoint without improvement |
| `train.py` is agent-modifiable | Only lines 50+ (outside DANGER ZONE) |
| `prepare.py` is read-only | `data/` is a DANGER ZONE |

## Why This BOUND Works

ML experimentation is inherently iterative — the agent tries many approaches, most fail. The BOUND constrains the cost of failure (5-minute budget) while protecting what matters (best model, validation data, experiment logs). The agent is free to experiment wildly within these boundaries.
