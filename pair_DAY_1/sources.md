# Sources — Day 1

## Canonical Papers

### 1. Zheng et al. (2023) — Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena
- **URL:** https://arxiv.org/abs/2306.05685
- **Why load-bearing:** This is the primary paper that documented verbosity bias and position bias in pairwise LLM judges. Introduced the MT-Bench framework and rigorously measured how judge preferences shift with response length. The verbosity bias finding (judges favor longer responses at length ratios above ~1.5x) is the mechanistic foundation for the detection methods in this explainer.

### 2. Hu et al. (2024) — Explaining Length Bias in LLM-Based Preference Evaluations
- **URL:** https://arxiv.org/abs/2407.01085
- **Why load-bearing:** Provides the theoretical decomposition of win rate into desirability (length-independent quality) and information mass (length-dependent content volume). This is the framework that explains *why* length leaks into judge scores — not just that it does. The AdapAlpaca method introduced here is the conceptual basis for the length-controlled swap test in this explainer.

## Tool Used

### scipy.stats.pointbiserialr
- **What it is:** Standard statistical function from the scipy library for computing point-biserial correlation between a continuous variable and a binary outcome.
- **How it was used:** Used to compute the correlation between email word count and judge outcome (chosen=1, rejected=0) in the detection code demonstration.
- **Install:** `pip install scipy` (standard library, no special setup)

## Additional References Consulted
- CALM framework (arxiv.org/abs/2410.02736) — comprehensive taxonomy of 12 LLM judge bias types
- Evaluating and Mitigating LLM-as-a-judge Bias in Communication Systems (arxiv.org/abs/2510.12462) — finding that detailed rubrics improve judge robustness to verbosity bias