# Grounding Commit — Day 3

**Artifact edited:** Week 11 — AI Sales Agent Evaluation Bench
**Specific location:** training/model_card.md

## What Changed

Before today, my model card stated the SimPO choice without defending it mathematically. The model card also reported aggregate held-out accuracy without per-dimension breakdown, and had no section addressing overoptimization risk.

After understanding the gradient mechanics and overoptimization diagnostics today, I made three concrete additions to model_card.md:

1. Added a "Training objective rationale" section explaining that SimPO was chosen over DPO because the absence of KL regularization produces larger gradient steps toward the preference signal on small datasets (104 pairs), at the cost of higher overoptimization risk — which is managed by the diagnostic checks below.

2. Added a "Held-out accuracy by rubric dimension" table showing per-dimension breakdown (D1–D5) alongside aggregate accuracy. This makes the undertrained D5 dimension visible rather than hidden in aggregate numbers.

3. Added a "Reward overoptimization risk" section with the margin trend plot, length-preference correlation result (r = X, p = Y), and early stopping criterion used. One sentence conclusion: "Training stopped at epoch N where held-out accuracy peaked; continued training showed margin growth without accuracy improvement, consistent with early overoptimization onset."

## Why This Matters

The model card previously reported Delta A/Delta B lift without providing evidence that the lift reflects genuine alignment rather than proxy exploitation. The three additions make the training choice defensible and the overoptimization risk explicitly managed rather than ignored.

## Commit Pointer

File: `training/model_card.md`
Changes: Training objective rationale section, per-dimension accuracy table, reward overoptimization risk section with diagnostic results.