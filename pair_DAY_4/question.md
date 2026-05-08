# Day 4 Question — Base Model Pre-training Contamination

## Final Sharpened Question

My Week 11 eval bench passes three contamination checks — n-gram overlap, embedding similarity, and time-shift verification — verifying that held-out tasks did not leak into my training partition. But if a client asked "how do you know Qwen didn't see these tasks during pre-training?" I have no answer. I cannot explain what membership inference tests exist for base model contamination, what they can actually prove, and why n-gram overlap on held-out partitions is a fundamentally different problem from pre-training data contamination. This gap means my benchmark's validity claim is incomplete.

**Artifact:** datasheet.md and the contamination section of my Week 11 benchmark documentation.

**What closing this gap changes:** I will be able to add an honest limitations section to datasheet.md that distinguishes what my contamination checks prove from what they cannot prove, and flag pre-training contamination as an open risk rather than a solved problem.

## Four-Property Self-Check

| Property | Status |
|---|---|
| Diagnostic | ✅ Distinguishes two different contamination problems — held-out leakage vs pre-training |
| Grounded in work | ✅ Tied to specific contamination claims in datasheet.md |
| Generalizable | ✅ Every FDE publishing a benchmark faces this exact challenge |
| Resolvable in one explainer | ✅ One explainer on contamination test limits closes it |
