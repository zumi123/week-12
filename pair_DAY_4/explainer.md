# Is Your Eval Defensible? Statistical Significance for Small Binary-Outcome Benchmarks

*Written for an engineer comparing v0.2 vs v0.3 pass rates on 89 held-out transcripts, where the improvement is 3 failures → 0 failures (3.4 percentage points), and who needs to know whether that difference is statistically meaningful or sampling noise.*

---

## The Question That Matters

Your v0.2 judge fails 3 of 89 held-out transcripts on the `regex_positive` criterion. Your v0.3 fix is expected to bring that to 0 failures. That is a 3.4 percentage-point improvement — from 96.6% to 100% pass rate.

Before you write "v0.3 is significantly better" in `methodology.md`, you need to answer: is that improvement real, or is it within the margin of error for a sample of 89 binary outcomes?

This is not a formality. A 3.4pp difference on 89 examples is genuinely ambiguous without a confidence interval. Here is how to compute one and what it tells you.

---

## The Load-Bearing Mechanism: Why Wald Fails Here

The intuitive CI formula — p̂ ± 1.96 × √(p̂(1-p̂)/n) — is the Wald interval. It is what most engineers reach for first. It is also the wrong choice here, for two reasons.

First, your pass rates are near boundary values (96.6% and 100%). The Wald interval assumes a normal approximation to the binomial distribution, which breaks down near 0 and 1. At p̂ = 1.0 (zero failures), Wald produces a CI of [1.0, 1.0] — a zero-width interval that is mathematically meaningless and statistically misleading.

Second, your sample is small (n=89). The normal approximation requires n to be large enough that both np̂ and n(1-p̂) exceed ~5. At p̂ = 0.966, n(1-p̂) = 89 × 0.034 = 3.0 — below that threshold.

The correct method for small samples near boundary values is the **Wilson interval** for single proportions and the **Wilson score test** for comparing two proportions.

---

## Method 1 — Wilson CI for Each Pass Rate

The Wilson interval inverts the score test rather than the Wald test, giving better coverage near boundaries.

```python
from statsmodels.stats.proportion import proportion_confint
import numpy as np

# v0.2: 86 pass, 3 fail, n=89
n = 89
k_v02 = 86
k_v03 = 89  # 0 failures

# Wilson (score) confidence intervals
ci_v02 = proportion_confint(k_v02, n, alpha=0.05, method='wilson')
ci_v03 = proportion_confint(k_v03, n, alpha=0.05, method='wilson')

print(f"v0.2 pass rate: {k_v02/n:.3f} | 95% CI: [{ci_v02[0]:.3f}, {ci_v02[1]:.3f}]")
print(f"v0.3 pass rate: {k_v03/n:.3f} | 95% CI: [{ci_v03[0]:.3f}, {ci_v03[1]:.3f}]")
```

**Expected output:**
```
v0.2 pass rate: 0.966 | 95% CI: [0.908, 0.989]
v0.3 pass rate: 1.000 | 95% CI: [0.959, 1.000]
```

Note that the Wilson CI for 100% pass rate is not zero-width — it correctly reflects the uncertainty of observing zero failures in a finite sample. The lower bound of ~0.959 means you can claim "with 95% confidence, the true pass rate is above 95.9%", not "the pass rate is exactly 100%."

---

## Method 2 — Is the Difference Significant? Fisher's Exact Test

To test whether v0.3 is significantly better than v0.2, you need a two-proportion test. For small samples with binary outcomes, Fisher's exact test is the correct choice — it makes no normal approximation and works correctly with small cell counts.

```python
from scipy.stats import fisher_exact
import numpy as np

# Contingency table:
# rows = version (v0.2, v0.3)
# cols = outcome (pass, fail)
table = np.array([
    [86, 3],   # v0.2: 86 pass, 3 fail
    [89, 0]    # v0.3: 89 pass, 0 fail
])

odds_ratio, p_value = fisher_exact(table, alternative='less')
# alternative='less': tests whether v0.2 has a lower pass rate than v0.3

print(f"Fisher's exact test: p = {p_value:.4f}")
print(f"Interpretation: {'significant at p<0.05' if p_value < 0.05 else 'not significant at p<0.05'}")
```

**Expected output:**
```
Fisher's exact test: p = 0.1211
Interpretation: not significant at p<0.05
```

This is the uncomfortable result: a 3-failure → 0-failure improvement on 89 examples does not reach conventional statistical significance. The p-value of ~0.12 means this difference could plausibly occur by chance even if v0.3 and v0.2 were identical in underlying quality.

---

## Why This Result Makes Sense

Three failures out of 89 is a small absolute number. The binomial distribution at p=0.966 assigns non-trivial probability to observing 0, 1, 2, or 3 failures in a sample of 89 — meaning zero failures is not strong evidence of a meaningfully different underlying pass rate.

To reach p < 0.05 for a 3.4pp improvement, you would need roughly 250-300 examples. This is not a flaw in your eval — it is the honest statistical reality of small benchmarks.

The right way to report this in `methodology.md` is:

> "v0.3 reduced `regex_positive` failures from 3/89 to 0/89 (96.6% → 100.0%, +3.4pp). Wilson 95% CI for v0.3: [95.9%, 100.0%]. Fisher's exact test: p = 0.12 (n.s. at α=0.05). The directional improvement is consistent with the LoRA target fix but does not reach statistical significance on this sample size; n ≈ 250 would be required for this effect size."

This is a stronger claim than "v0.3 is better" — because it is honest about what the data can and cannot prove.

---

## When Would You Use Bootstrap Instead?

Bootstrap resampling is appropriate when you are comparing aggregate scores across many tasks (like your Delta A / Delta B lift) rather than a single binary pass rate. For a single proportion, Wilson and Fisher's exact are analytically exact and preferable to bootstrap for small n.

Use bootstrap when:
- Your metric is a mean over continuous scores (not a binary pass rate)
- You want to compare two agents across the same task set (paired bootstrap)
- Your test statistic is complex enough that an analytical distribution is unavailable

For your specific v0.2 vs v0.3 comparison on `regex_positive`, Fisher's exact is the right tool.

---

## What to Add to Your Artifacts

In `methodology.md`, replace the directional comparison with:
1. Wilson CIs for both v0.2 and v0.3 pass rates
2. Fisher's exact test p-value with explicit interpretation
3. Sample size required for significance at this effect size
4. One honest sentence: "This improvement is directionally consistent with the fix but does not reach statistical significance on n=89; treat as a strong signal requiring validation on a larger held-out set."

---

## Sources

- **Brown, Cai & DasGupta (2001)** — Interval Estimation for a Binomial Proportion. The canonical statistical paper establishing that Wald intervals perform poorly near boundary values and recommending Wilson and Agresti-Coull intervals for small samples. *Statistical Science*, 16(2), 101–133. https://www.jstor.org/stable/2676784

- **statsmodels documentation** — `proportion_confint` with `method='wilson'` and `fisher_exact` from scipy.stats. Both standard Python statistics libraries. https://www.statsmodels.org/stable/generated/statsmodels.stats.proportion.proportion_confint.html
