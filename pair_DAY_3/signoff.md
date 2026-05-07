# Sign-off — Day 3

**Gap:** SimPO gradient mechanics vs DPO — why removing the reference model produces more movement toward the preference signal on small datasets, and what that means for overoptimization risk.
**Sign-off status:** ✅ Closed

## What I Understand Now That I Did Not Before

Before today I could state that SimPO works better on small datasets because it removes the reference model. I could not explain why that is true at the gradient level.

The explainer I received closed this gap. DPO's loss function includes an implicit KL divergence term between the trained policy and a frozen reference model — this regularizes how far the policy can drift. On a small dataset, this constraint limits the adapter's ability to move toward the preference signal because the gradient update is constantly pulled back toward the reference distribution.

SimPO removes this constraint entirely. The loss directly maximizes the margin between average per-token log-probability of chosen vs rejected responses, normalized for length. With no KL term resisting the update, the gradient steps are larger and more directed toward the preference signal — which is why SimPO converges faster on small datasets.

The cost is that the same absence of KL regularization means the adapter can drift further from the base distribution in ways that exploit proxy patterns rather than genuine quality distinctions. That is the overoptimization risk I could not previously articulate.

I can now write a model card section that defends the SimPO choice with this reasoning and includes the overoptimization diagnostic checks as evidence that the risk was managed.