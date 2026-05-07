# Tweet Thread — Day 3

---

**Tweet 1**
You trained a judge adapter on preference pairs. Held-out accuracy improved. But how do you know you're actually aligning the judge — and not just overfitting to surface patterns in your rubric?

Here's how to tell the difference. 

---

**Tweet 2**
The mechanism — why overoptimization happens:

Your preference training optimizes a proxy reward (the model's preference predictions) as a stand-in for true quality.

Gao et al. (2023) showed this relationship: as you optimize against the proxy, gold reward initially improves — then degrades past a threshold.

On small datasets (104 pairs), the model has few examples of genuine quality distinctions and many chances to latch onto spurious patterns: response length, formatting, keyword frequency.

---

**Tweet 3**
The 4 diagnostic checks — run these before you ship:

1. Margin trend: plot training margin vs held-out accuracy across epochs. If margin keeps rising while accuracy plateaus → overoptimizing.

2. Length correlation: point-biserial r between word count and preference label. |r| > 0.3 means your judge is scoring length.

3. Dimension slices: break held-out accuracy by rubric dimension. Aggregate lift hides undertrained dimensions.

4. Cross-judge disagreement: compare trained vs base model on held-out pairs. High disagreement on easy pairs = overfit signal.

---

**Tweet 4**
Why SimPO is higher risk than DPO for overoptimization:

DPO has an implicit KL regularization term — the reference model constrains how far the policy can drift. This slows overoptimization.

SimPO removes this constraint entirely. It optimizes the margin directly. This is why it moves faster on small datasets — and why it requires more careful monitoring.

Without KL regularization, the only thing stopping collapse onto surface heuristics is your early stopping criterion and held-out evaluation.

---

**Tweet 5**
What to add to your model card:

→ Margin trend plot across epochs
→ Per-dimension accuracy breakdown (not just aggregate)
→ Length-preference correlation coefficient
→ One sentence: "Training stopped at epoch N where held-out accuracy peaked at X%; continued training showed [margin growth / accuracy plateau]."

The difference between "we trained a judge" and "we have a judge we can defend" is whether you ran these checks.

---

**Tweet 6**
Full explainer with runnable diagnostic code, the four checks, and what real alignment looks like vs overoptimization — with canonical sources:


Sources:
→ Gao et al. 2023 (Scaling Laws for Reward Model Overoptimization): arxiv.org/abs/2210.10760
→ Huang et al. 2024 (KL-Regularization Mythos): arxiv.org/abs/2407.13399