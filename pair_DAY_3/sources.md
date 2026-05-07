# Sources — Day 3

## Canonical Papers

### 1. Gao et al. (2023) — Scaling Laws for Reward Model Overoptimization
- **URL:** https://arxiv.org/abs/2210.10760
- **Why load-bearing:** The canonical paper establishing that optimizing against a proxy reward model diverges from true gold reward past a measurable threshold. Introduces the proxy vs gold reward framework, empirical scaling laws for when overoptimization occurs, and the role of KL penalty in slowing (but not eliminating) the effect. The four diagnostic checks in this explainer are grounded in the signals this paper identified.

### 2. Huang et al. (2024) — Correcting the Mythos of KL-Regularization
- **URL:** https://arxiv.org/abs/2407.13399
- **Why load-bearing:** Establishes that KL regularization in DPO is theoretically insufficient to prevent overoptimization — the model can still drift away from preferred responses covered by the training data. Provides the theoretical basis for why SimPO, which removes KL regularization entirely, is higher risk for overoptimization than DPO on small datasets despite its faster convergence.

## Tool Used

### scipy.stats.pointbiserialr + matplotlib
- **What it is:** Standard Python statistical and plotting libraries
- **How it was used:** pointbiserialr for the length-preference correlation diagnostic; matplotlib for the margin-vs-held-out-accuracy trend plot. Both demonstrated with runnable code in the explainer.
- **Install:** Both included in standard scientific Python stack (pip install scipy matplotlib)