# Canonical List — Week 12

*Annotated papers, tools, and engineering patterns worth every Forward-Deployed Engineer reading.*

---

## Papers

### 1. Zheng et al. (2023) — Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena
**URL:** https://arxiv.org/abs/2306.05685
**Why read it:** The foundational paper on LLM-as-a-judge evaluation. Documents verbosity bias, position bias, and self-preference bias with empirical measurements. Essential reading before building any eval bench that uses an LLM judge. The MT-Bench framework is still the reference point for pairwise judge evaluation.

### 2. Hu et al. (2024) — Explaining Length Bias in LLM-Based Preference Evaluations
**URL:** https://arxiv.org/abs/2407.01085
**Why read it:** Provides the theoretical decomposition of judge win rate into desirability (length-independent quality) and information mass (length-dependent content volume). Explains mechanically why judges conflate the two. Read alongside Zheng et al. for a complete picture of judge bias.

### 3. Patil et al. (2023) — Gorilla: Large Language Model Connected with Massive APIs
**URL:** https://arxiv.org/abs/2305.15334
**Why read it:** The foundational paper on training LLMs to call real-world APIs reliably. Demonstrates empirically that tool description quality directly and measurably affects call accuracy. Essential reading before writing tool schemas for any production agent.

### 4. Gao et al. (2023) — Scaling Laws for Reward Model Overoptimization
**URL:** https://arxiv.org/abs/2210.10760
**Why read it:** The canonical paper on proxy reward optimization diverging from gold reward. Establishes the empirical scaling laws for when overoptimization occurs and introduces the proxy vs gold reward framework. Essential reading before deploying any preference-trained model.

### 5. Huang et al. (2024) — Correcting the Mythos of KL-Regularization
**URL:** https://arxiv.org/abs/2407.13399
**Why read it:** Shows that KL regularization in DPO is theoretically insufficient to prevent overoptimization. Provides the theoretical basis for why SimPO, which removes KL regularization, requires more careful monitoring on small datasets despite faster convergence.

### 6. Brown, Cai & DasGupta (2001) — Interval Estimation for a Binomial Proportion
**URL:** https://www.jstor.org/stable/2676784
**Why read it:** The canonical statistics paper establishing that Wald confidence intervals perform poorly near boundary values and for small samples. Every engineer reporting pass rates on eval benchmarks should know the Wilson interval. Short, readable, and directly applicable.

---

## Tools and Engineering Patterns

### 1. scipy.stats — pointbiserialr, fisher_exact, bootstrap
**What it is:** Standard Python statistical library
**When to use it:** Length bias detection (pointbiserialr), two-proportion significance testing on small samples (fisher_exact), agent benchmark confidence intervals (bootstrap). The first tool to reach for when validating any eval metric claim.

### 2. statsmodels — proportion_confint (method='wilson')
**What it is:** Python statistics library with Wilson CI implementation
**When to use it:** Any time you are reporting a pass rate near 0% or 100% on a small sample. The Wilson interval gives correct coverage where Wald fails. One-liner to add to any benchmark reporting script.

### 3. Anthropic tool_use content block + stop_reason pattern
**What it is:** The canonical agent loop pattern keyed on stop_reason values
**When to use it:** Every time you build an agent loop. Check stop_reason exhaustively (end_turn, tool_use, max_tokens). Always return a tool_result even on exception with is_error: True. Add completed-step tracking to the system prompt. These three patterns prevent the most common agent looping failures.

### 4. Prefix caching prompt structure
**What it is:** Engineering pattern for structuring prompts to maximize KV cache hit rate
**When to use it:** Any batch inference system with repeated fixed context. Fixed content (system prompt, context files, rubrics) must come first in the prompt, byte-identical across all calls in a session, and exceed the minimum cacheable length (1,024 tokens on Anthropic). Reduces prefill cost to ~10% on cache hits.

### 5. Per-dimension eval breakdown
**What it is:** Reporting pattern for preference-trained judge evaluation
**When to use it:** Always — never report only aggregate held-out accuracy for a multi-rubric judge. Break down accuracy by rubric dimension. Aggregate lift hides undertrained dimensions and can mask overoptimization on a subset of the preference surface.