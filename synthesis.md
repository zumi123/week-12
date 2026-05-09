# Week 12 Synthesis

**Zemzem Hibet | Tenacious Recruitment Program (TRP) — Week 12**

---

## The Ten Gaps Closed

### Gaps I Named (Questions I Asked)

**Day 1 — KV Cache Mechanism**
I knew prefix caching worked and could verify it from API metadata, but I could not explain what keys and values are at the transformer level, how many bytes per token per layer they occupy, or why recomputing them is expensive enough to produce measurable cost savings. Closing this gap changed how I structured the batch inference loop in my Week 10 system — the fixed prefix now comes first in every call, byte-identical across all 50 leads, which is a mechanistic requirement I can now defend rather than a style choice.

**Day 2 — Agent Loop Failure Modes**
I had two real looping traces (19d13ac9, 293b3bbb) from my Week 10 agent — 682 seconds each, terminated by user_stop — and no framework for diagnosing why. I could not explain what happens at the token level when a model decides to call a tool. Closing this gap gave me three specific failure modes to check: truncated tool calls (max_tokens mid-JSON), missing tool_result returns on exception, and planning loops without completed-step tracking.

**Day 3 — SimPO vs DPO Gradient Mechanics**
I chose SimPO over DPO for my Week 11 judge training because "it works better on small datasets" — but I could not derive why at the gradient level. Closing this gap gave me the actual mechanism: SimPO removes the KL regularization term that resists policy drift, producing larger gradient steps toward the preference signal on small datasets — at the cost of higher overoptimization risk without careful monitoring.

**Day 4 — Contamination Test Limits**
My Week 11 benchmark passed three contamination checks and I presented this as evidence the benchmark is uncontaminated. I could not explain what those checks actually prove versus what they cannot. Closing this gap forced an honest rewrite of my datasheet.md contamination section — distinguishing held-out partition integrity (what my checks prove) from base model pre-training contamination (an open risk I cannot rule out).

### Gaps I Researched (Questions My Partners Asked)

**Day 1 — LLM Judge Length Bias Detection**
Partner's question: how to detect and quantify length bias in Qwen3-Next-80B given chosen emails averaging 108 words and rejected averaging 231 words. Researched and explained the point-biserial correlation method and the length-controlled swap test. The key insight: a significant length-preference correlation does not prove bias if short emails genuinely perform better — the swap test separates these by holding content constant while varying length.

**Day 2 — Function-Calling Token Level Mechanics**
Partner's question: what happens at the token level when a model decides to call a tool — what does the raw output look like, how was the model trained to produce it. Researched and explained the tool_use content block structure, stop_reason values, and the three looping failure modes. The canonical agent loop pattern with explicit stop_reason handling was the concrete artifact that closed the gap.

**Day 3 — Reward Model Overoptimization Diagnostics**
Partner's question: how to diagnose whether LoRA preference training is learning stable winner-vs-loser behavior versus exploiting shortcuts. Researched and explained four concrete checks: margin trend across epochs, length-preference correlation, per-dimension accuracy breakdown, and cross-judge disagreement rate. The key insight: aggregate held-out lift hides undertrained rubric dimensions.

**Day 4 — Statistical Significance for Small Binary Benchmarks**
Partner's question: whether a 3-failure to 0-failure improvement on 89 examples is statistically meaningful. Researched and explained why Wald intervals fail near boundary values, the Wilson CI as the correct alternative, and Fisher's exact test for two-proportion comparison. The uncomfortable result — p = 0.12, not significant — was the specific finding that closed the gap and forced an honest methodology claim.

---

## Most Surprising Thing I Learned

The most surprising finding of the week was that a 3-failure to 0-failure improvement on 89 held-out examples — which looks like a clear win — does not reach statistical significance at p < 0.05. Fisher's exact test gives p = 0.12. This is not a flaw in the benchmark; it is the honest statistical reality of small binary-outcome eval sets. You need approximately 250-300 examples to detect a 3.4 percentage-point effect. Most AI engineers I know (including myself before this week) would have reported that improvement as a result without running the test.

---

## Canonical Reading List

See `canonical_list.md` for the full annotated list.

---

## Portfolio Impact

See `portfolio_update.md` for the hiring-manager-facing summary of how the four grounding commits improve the Week 10 and Week 11 portfolio.