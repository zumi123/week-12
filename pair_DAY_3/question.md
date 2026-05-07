# Day 3 Question — SimPO Gradient Mechanics vs DPO

## Final Sharpened Question

In my Week 11 eval bench I trained a SimPO judge on 104 preference pairs and chose SimPO over DPO partly because it removes the reference model and works better on small datasets. I can state that intuition but I cannot derive why — specifically, what the SimPO gradient actually penalizes at the parameter level, how that differs from the DPO gradient, and why removing the reference model mathematically produces more movement toward the preference signal on small datasets rather than just less regularization. Without the gradient derivation, I am defending an engineering choice I cannot fully explain.

**Artifact:** model_card.md and the SimPO training script in Week 11.

**What closing this gap changes:** I will be able to write a model card section that defends the SimPO choice with the actual mathematical reasoning — not just "SimPO works better on small datasets" but why, at the gradient level, that is true.

## Four-Property Self-Check

| Property | Status |
|---|---|
| Diagnostic | ✅ Names the specific mechanism gap: gradient derivation, parameter-level penalization, reference model role |
| Grounded in work | ✅ Tied to SimPO training script and model_card.md in Week 11 |
| Generalizable | ✅ Every FDE training a preference-optimized judge faces this choice |
| Resolvable in one explainer | ✅ One mechanism, one blog post closes it |