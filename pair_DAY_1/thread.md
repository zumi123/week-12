# Tweet Thread — Day 1

---

**Tweet 1**
You built an LLM judge to evaluate AI-generated emails. Chosen emails: 108 words. Rejected: 231 words. How do you know the judge is scoring persuasive quality — and not just rewarding brevity? Here's how to find out.

---

**Tweet 2**
Why length leaks into judge scores:

LLM judges do next-token prediction over both responses. A longer response carries more tokens, more surface signals of effort, more chances to hit rubric keywords.

Researchers decompose judge preference into two components:
→ Desirability (quality, independent of length)
→ Information mass (content volume, scales with length)

Judges conflate the two. That's the bug.

---

**Tweet 3**
The 5-minute audit — run this on your eval data:

```python
from scipy import stats

lengths = chosen_lengths + rejected_lengths
labels = [1]*len(chosen_lengths) + [0]*len(rejected_lengths)

r, p = stats.pointbiserialr(lengths, labels)
```

r < 0 means shorter emails win more.
|r| > 0.3 and p < 0.05 = length is a significant predictor.

This tells you IF length predicts outcome. It doesn't tell you WHY.

---

**Tweet 4**
The definitive test — the length-controlled swap:

Take 10 pairs where the judge chose the shorter email.
Pad the shorter email with filler to match the rejected length.
Same persuasive content, more words.

Re-run the judge.

If the padded version now wins → verbosity bias confirmed.
If the original still wins → judge is tracking quality, not length.

---

**Tweet 5**
The honest complication: in cold email outreach, short often IS better.

Your judge preferring 108-word emails might be correct, not biased.

The swap test separates these. If the judge scores the original content higher even when padded to match the rejected length → that's the judge working correctly, not a bug.

Report both numbers: point-biserial r AND swap test result. One without the other is incomplete.

---

**Tweet 6**
Full explainer with runnable code, the desirability/information-mass decomposition, and what to add to your datasheet.md:


Sources:
→ Zheng et al. 2023 (MT-Bench) arxiv.org/abs/2306.05685
→ Hu et al. 2024 (Length Bias Decomposition) arxiv.org/abs/2407.01085