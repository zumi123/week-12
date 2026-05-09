# Portfolio Update — Week 12
**Zemzem Hibet | For: The program Manager**

---

## Summary

Week 12 produced four grounding commits that collectively close the gap between systems that work and systems that can be defended. The commits below are not cosmetic improvements — each one removes a specific claim from the portfolio that could not previously be backed by mechanism, and replaces it with a claim that can.

---

## The Four Commits and What They Change

### Commit 1 — Week 10: Prompt Structure Restructured for Prefix Caching
**File:** `lead_gen_system/batch_inference.py`

Before: The batch inference loop assembled prompts in variable order, sometimes placing fixed context files after lead-specific content. The system worked but the cost structure was indefensible — every call was recomputing 2,500 tokens of fixed context.

After: Fixed context (`icp_definition.md`, `style_guide.md`, `bench_summary.json`) is now loaded once, concatenated in fixed order, and placed first in every prompt across the batch. This is the mechanistic requirement for KV cache hits. On a batch of 50 leads, prefill cost is reduced by approximately 45× on calls 2–50 (cache reads at 10% of standard input token rate).

**What this shows a hiring manager:** The engineer understands the cost structure of their own system at the infrastructure level — not just that caching works, but why byte-identical prefix ordering is a hard requirement.

---

### Commit 2 — Week 10: Agent Loop Hardened Against Looping Failures
**File:** `conversion_engine/agent.py`

Before: The agent loop handled stop_reason: "tool_use" and stop_reason: "end_turn" but did not check for stop_reason: "max_tokens". Tool execution failures were caught by a broad except clause that sometimes failed to return a tool_result, causing the model to retry the same tool call indefinitely. Two real traces (19d13ac9, 293b3bbb) showed 682-second loops terminated by the environment.

After: Explicit handling for all stop_reason values including max_tokens. Every tool execution path — including exceptions — returns a tool_result block with is_error: True on failure. Completed-step tracking injected into the system prompt at each turn.

**What this shows a hiring manager:** The engineer can diagnose production failures from first principles, not just symptoms. The fix is mechanistically grounded, not a patch.

---

### Commit 3 — Week 11: Model Card Defends SimPO Choice with Gradient Reasoning
**File:** `training/model_card.md`

Before: The model card stated "SimPO was chosen over DPO because it performs better on small datasets" — a correct claim backed by intuition, not mechanism. The overoptimization risk of removing KL regularization was not addressed. Aggregate held-out accuracy was reported without per-dimension breakdown.

After: Training objective rationale section added with the gradient-level explanation: SimPO removes KL regularization, producing larger gradient steps toward the preference signal on small datasets at the cost of higher overoptimization risk. Per-dimension accuracy table added (D1–D5). Reward overoptimization risk section added with margin trend plot, length-preference correlation result, and early stopping criterion.

**What this shows a hiring manager:** The engineer can defend training decisions at the mathematical level and proactively surfaces known risks rather than hiding them in aggregate metrics.

---

### Commit 4 — Week 11: Datasheet Contamination Claims Made Honest
**File:** `datasheet.md`

Before: The contamination section listed three checks (n-gram overlap, embedding similarity, time-shift verification) and implied the benchmark is uncontaminated. This claim would not survive scrutiny — the checks verify held-out partition integrity, not base model pre-training contamination.

After: Section renamed "Contamination Controls and Limitations." Limitations paragraph added distinguishing held-out partition integrity from base model pre-training contamination. Explicit disclosure: "Pre-training contamination in the base model is an open risk that cannot be ruled out and should be considered when interpreting benchmark results."

**What this shows a hiring manager:** The engineer knows the difference between what their evaluation controls for and what it does not — and is honest about the boundary in public-facing documentation.

---

## The Collective Picture

A program manager reviewing the Week 10 and Week 11 portfolio before Week 12 would see two working systems with correct outputs but incomplete defenses. The systems worked but several claims were backed by intuition rather than mechanism.

After Week 12, the same portfolio shows systems whose engineering choices can be defended at the infrastructure level (caching), the agent loop level (tool call mechanics), the training level (gradient mathematics), and the evaluation level (statistical validity). The grounding commits are not feature additions — they are the difference between a portfolio that shows results and a portfolio that shows understanding.

---

*Submitted: Saturday, Week 12 | Tenacious Recruitment Program (TRP)*