# Are You Actually Aligning Your Judge, or Just Overfitting to It?
## Diagnosing Reward Overoptimization in LoRA Preference Training

*Written for an engineer who trained a small LoRA judge adapter with preference optimization, reported held-out lift, but cannot defend whether that lift reflects genuine alignment or early reward overoptimization on a narrow preference surface.*

---

## The Question That Matters

You trained a judge adapter on preference pairs. Your held-out metrics improved. But here is the uncomfortable question: is your judge actually better at evaluating sales agent quality, or did it learn to exploit surface patterns in your rubric — response length, keyword presence, structural formatting — that correlate with your labels but do not generalize?

This is reward overoptimization. It is not a theoretical risk. It is the most common failure mode in small-dataset preference training, and it is detectable if you know what to measure.

---

## The Load-Bearing Mechanism: What Overoptimization Actually Is

When you train a judge with preference optimization, you are optimizing a proxy reward signal — the model's preference predictions — as a stand-in for the true quality signal you actually care about (whether the judge correctly identifies better sales responses). The proxy is imperfect by construction: it was trained on a small, domain-specific dataset with a narrow preference surface.

Gao et al. (2023) — the canonical paper on this phenomenon — showed that as you optimize against a proxy reward model, the gold reward (true quality) initially improves but then degrades past a certain threshold. This is Goodhart's Law applied to alignment: the proxy becomes the target, and the model learns to satisfy the proxy in ways that diverge from the true signal.

In your specific setup, the risk is amplified by three factors:

**Small dataset.** Your 104 preference pairs define a narrow preference surface. The model has few examples from which to learn genuine quality distinctions and many opportunities to latch onto spurious correlations — response length differences, formatting patterns, rubric keyword frequency.

**Skewed label distribution.** With 32 pairs on D4 and only 3 on D5, your adapter is undertrained on the tone dimension by construction. A model that scores well on held-out D4 pairs but fails on D5 has not generalized — it has memorized a subset of your preference surface.

**No reference model (SimPO).** SimPO removes the KL regularization term that constrains how far the policy can drift from the base model. This is why SimPO moves faster on small datasets — but it also means there is less resistance to overoptimization. The model can move further from the base distribution in fewer steps.

---

## How to Diagnose It: Four Concrete Checks

### Check 1 — Margin trend across epochs

In SimPO, the training objective maximizes the margin between the average log-probability of chosen and rejected responses. Plot this margin across training epochs.

```python
import json
import matplotlib.pyplot as plt

# Load your training logs
with open("ablations/ablation_results.json") as f:
    results = json.load(f)

epochs = [r["epoch"] for r in results["training_log"]]
margins = [r["chosen_logprob_avg"] - r["rejected_logprob_avg"] for r in results["training_log"]]
held_out = [r["held_out_accuracy"] for r in results["training_log"]]

fig, ax1 = plt.subplots()
ax2 = ax1.twinx()
ax1.plot(epochs, margins, 'b-', label='training margin')
ax2.plot(epochs, held_out, 'r-', label='held-out accuracy')
plt.title("Margin vs held-out accuracy across epochs")
plt.savefig("overoptimization_check.png")
```

**What to look for:** If training margin keeps rising while held-out accuracy plateaus or drops, you are overoptimizing. The margin is the proxy; held-out accuracy is closer to gold. Divergence between them is the signal.

### Check 2 — Length correlation audit

Run the same point-biserial correlation from Day 1 on your preference pairs. If your judge's preference correlates with response length (|r| > 0.3, p < 0.05), length is a confound in your training signal.

```python
from scipy import stats

chosen_lengths = [len(p["chosen"].split()) for p in preference_pairs]
rejected_lengths = [len(p["rejected"].split()) for p in preference_pairs]

lengths = chosen_lengths + rejected_lengths
labels = [1]*len(chosen_lengths) + [0]*len(rejected_lengths)

r, p = stats.pointbiserialr(lengths, labels)
print(f"Length-preference correlation: r={r:.3f}, p={p:.4f}")
```

If your chosen responses are systematically longer or shorter than rejected ones, your adapter may be learning a length heuristic rather than quality distinctions.

### Check 3 — Rubric dimension slice analysis

Your benchmark has five rubric dimensions (D1–D5). Report held-out accuracy broken down by dimension, not just in aggregate.

```python
by_dimension = {}
for pair in held_out_pairs:
    dim = pair["rubric_dimension"]
    correct = pair["judge_prediction"] == pair["gold_label"]
    by_dimension.setdefault(dim, []).append(correct)

for dim, scores in by_dimension.items():
    n = len(scores)
    acc = sum(scores) / n
    print(f"{dim}: {acc:.2f} accuracy ({n} pairs)")
```

**What to look for:** If D5 accuracy is near chance (0.50) while D1 and D4 accuracy is high, your adapter has not generalized — it has fitted to the overrepresented dimensions. Aggregate held-out lift hides this.

### Check 4 — Cross-judge disagreement check

Run your held-out pairs through both your trained adapter AND the base model (before adaptation). Count disagreements.

```python
agreements = 0
disagreements = 0

for pair in held_out_pairs:
    trained_pred = trained_judge.predict(pair)
    base_pred = base_judge.predict(pair)
    if trained_pred == base_pred:
        agreements += 1
    else:
        disagreements += 1

disagreement_rate = disagreements / (agreements + disagreements)
print(f"Disagreement rate: {disagreement_rate:.2f}")
```

A high disagreement rate (> 0.30) on held-out pairs means your adapter has moved substantially from the base model's judgment. This is not necessarily bad — but it warrants inspection. For each disagreement, check whether the trained adapter or the base model is correct. If the base model is frequently right where the adapter is wrong, you have overfit.

---

## What Real Alignment Looks Like vs Overoptimization

| Signal | Genuine alignment | Overoptimization |
|--------|------------------|------------------|
| Margin trend | Rises then plateaus | Keeps rising while held-out drops |
| Length correlation | Low (|r| < 0.2) | High (|r| > 0.3) |
| Dimension slices | Balanced improvement | High on frequent dims, near-chance on rare |
| Cross-judge disagreement | Concentrated on genuinely hard pairs | Spread across easy pairs too |

---

## What to Add to Your Artifacts

**In `training/train.py`:** Log chosen_logprob_avg and rejected_logprob_avg at every epoch alongside held-out accuracy. Add an early stopping check: if held-out accuracy drops two epochs in a row, stop training.

**In `ablations/ablation_results.json`:** Add a `robustness_slices` section with per-dimension held-out accuracy and the length-preference correlation coefficient.

**In `training/model_card.md`:** Add a section titled "Reward overoptimization risk" with:
- The margin trend plot
- Per-dimension accuracy breakdown
- Length correlation result
- One sentence: "Training was stopped at epoch N where held-out accuracy peaked at X%; continued training showed [margin growth / accuracy plateau] consistent with [early overoptimization / stable alignment]."

---

## The Adjacent Concept: KL Regularization and Why SimPO Skips It

DPO implicitly regularizes via a reference model — the KL divergence between the trained policy and the reference model appears in the loss, penalizing the policy from drifting too far from the base distribution. This is the mechanism that slows overoptimization in DPO.

SimPO removes this constraint entirely. It optimizes the margin directly, with only length normalization as a regularizer. This is why SimPO moves faster on small datasets — and why it requires more careful monitoring of the overoptimization signals above. Without KL regularization, the only thing stopping your adapter from collapsing onto surface heuristics is your held-out evaluation and your training stopping criterion.

---

## Sources

- **Gao et al. (2023)** — Scaling Laws for Reward Model Overoptimization. The canonical paper establishing that proxy reward optimization diverges from gold reward past a threshold, with empirical scaling laws for when this happens. Proceedings of ICML 2023. https://arxiv.org/abs/2210.10760

- **Huang et al. (2024)** — Correcting the Mythos of KL-Regularization: Direct Alignment without Overoptimization via Chi-Squared Preference Optimization. Establishes that KL regularization in DPO is insufficient to prevent overoptimization and proposes stronger guarantees. https://arxiv.org/abs/2407.13399

- **Tool used:** `scipy.stats.pointbiserialr` for length-preference correlation; `matplotlib` for margin trend visualization. Both standard library, no special setup required.