# Tweet Thread — Day 4

---

**Tweet 1**
Your v0.2 judge fails 3/89 eval tasks. Your v0.3 fix brings it to 0/89. That's a 3.4pp improvement.

Is it statistically meaningful — or sampling noise?

Here's how to find out, and why the answer might surprise you. 🧵

---

**Tweet 2**
First: don't use the Wald interval.

The standard p̂ ± 1.96√(p̂(1-p̂)/n) formula breaks down near boundary values (96.6%, 100%) and small n.

At p̂ = 1.0, Wald gives a zero-width CI — mathematically meaningless.

Use the Wilson interval instead:

from statsmodels.stats.proportion import proportion_confint
ci = proportion_confint(89, 89, alpha=0.05, method='wilson')
# → [0.959, 1.000]

The lower bound ~0.959 means: "with 95% confidence, true pass rate is above 95.9%" — not "it's exactly 100%."

---

**Tweet 3**
To test whether v0.3 is actually better than v0.2, use Fisher's exact test:

from scipy.stats import fisher_exact
table = [[86, 3],  # v0.2: 86 pass, 3 fail
         [89, 0]]  # v0.3: 89 pass, 0 fail
_, p = fisher_exact(table, alternative='less')
# p ≈ 0.12

Not significant at p < 0.05.

A 3-failure → 0-failure improvement on 89 examples doesn't clear the bar. The binomial distribution assigns real probability to observing 0 failures by chance even if nothing changed.

---

**Tweet 4**
Why this makes sense — and why it matters:

Three failures out of 89 is a small absolute count. You need ~250-300 examples to reach p < 0.05 for a 3.4pp effect.

This is not a flaw in your eval. It's the honest statistical reality of small benchmarks.

The right claim in your methodology.md:
"Directional improvement consistent with the fix. Fisher's exact p = 0.12 (n.s.). n ≈ 250 required for significance at this effect size."

That's stronger than "v0.3 is better" — because it's honest about what 89 examples can prove.

---

**Tweet 5**
When to use bootstrap instead:

Bootstrap is for aggregate scores over many tasks — like comparing mean Delta A / Delta B across an agent benchmark.

For a single binary pass rate on a small sample, Wilson + Fisher's exact are analytically exact and better than bootstrap.

Rule of thumb:
→ Binary pass rate, small n → Wilson CI + Fisher's exact
→ Mean score over many tasks → paired bootstrap CI

---

**Tweet 6**
Full explainer with runnable code, the Wilson CI calculation, Fisher's exact test, and what to write in your methodology.md so your improvement claim is statistically defensible:


Sources:
→ Brown, Cai & DasGupta (2001) — the canonical paper on Wald interval failure near boundaries: jstor.org/stable/2676784
→ statsmodels proportion_confint + scipy fisher_exact
