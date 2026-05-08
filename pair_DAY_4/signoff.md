# Sign-off — Day 4

**Gap:** Distinguishing held-out partition contamination checks from base model pre-training contamination — what my three checks actually prove and what they cannot.
**Sign-off status:** ✅ Closed

## What I Understand Now That I Did Not Before

Before today I presented my contamination checks — n-gram overlap, embedding similarity, and time-shift verification — as evidence that my benchmark is uncontaminated. I could not explain what that claim actually covers or where it breaks down.

The explainer I received closed this gap by drawing a clean line between two different problems.

The first problem — held-out partition leakage — is whether tasks from my held-out set leaked into my training partition during my own data preparation. My three checks address this directly and correctly. N-gram overlap verifies no verbatim task content crossed the partition boundary. Embedding similarity verifies no semantically near-identical tasks crossed it. Time-shift verification documents that data sources were collected before training began. These checks are valid and my benchmark passes them.

The second problem — base model pre-training contamination — is whether Qwen saw similar tasks during its own pre-training on internet-scale data before I ever touched it. My checks say nothing about this. The base model's training corpus is not available for inspection. Membership inference attacks exist for this problem but require white-box access to model weights and are computationally expensive — not something a benchmark author can routinely perform.

The honest claim is: my benchmark controls for held-out leakage within my pipeline. Pre-training contamination in the base model is an open risk that I cannot rule out and should disclose.
