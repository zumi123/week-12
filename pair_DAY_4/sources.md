# Sources — Day 4

## Canonical Papers

### 1. Brown, Cai & DasGupta (2001) — Interval Estimation for a Binomial Proportion
- **URL:** https://www.jstor.org/stable/2676784
- **Why load-bearing:** The canonical statistical paper establishing that Wald confidence intervals perform poorly near boundary values (p near 0 or 1) and for small samples. Recommends Wilson and Agresti-Coull intervals as superior alternatives. Directly load-bearing for the central claim of this explainer: that the Wald interval is the wrong choice for comparing 96.6% vs 100% pass rates on n=89.

### 2. statsmodels documentation — proportion_confint
- **URL:** https://www.statsmodels.org/stable/generated/statsmodels.stats.proportion.proportion_confint.html
- **Why load-bearing:** Authoritative documentation for the Wilson interval implementation used in the explainer's runnable code. Confirms that method='wilson' implements the correct score-test inversion and handles boundary values correctly.

## Tool Used

### scipy.stats.fisher_exact + statsmodels proportion_confint
- **What it is:** Standard Python statistical libraries
- **How it was used:** proportion_confint with method='wilson' for boundary-safe confidence intervals on single proportions; fisher_exact for two-proportion significance testing on small contingency tables. Both demonstrated with runnable code producing concrete output.
- **Install:** pip install scipy statsmodels
