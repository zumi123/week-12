# Day 1 Question — KV Cache Mechanics

## Final Sharpened Question

My Week 10 lead generation system (Automated Lead Generation and Conversion System for Tenacious Consulting and Outsourcing) sends the same `icp_definition.md`, `style_guide.md`, and `bench_summary.json` prefix — roughly 2,500 tokens — in every LLM call across a batch of 50 leads.

I know prefix caching works and I can verify it from API metadata (`usage.cache_read_input_tokens` and `usage.cache_creation_input_tokens` in Anthropic's response). But I cannot explain what is actually being stored in the KV cache at the transformer level: what keys and values are, how many bytes per token per layer they occupy, what GPU memory they consume, and why recomputing them from scratch is expensive enough that caching produces a measurable cost reduction.

Without understanding the mechanism, I cannot reason about why byte-for-byte prefix identity is a hard requirement, or defend the cost tradeoff in my system to a client.

## Artifact Pointer

**Week 10 system:** `Automated Lead Generation and Conversion System for Tenacious Consulting and Outsourcing`

Specifically: the batch inference loop that loads `icp_definition.md`, `style_guide.md`, and `bench_summary.json` as a fixed prefix before every lead-scoring and email-composition call.

**What closing this gap changes:** I will be able to explain to a client why restructuring the prompt so the fixed context comes first — and why keeping it byte-for-byte identical across calls — is not a style choice but a mechanistic requirement for cache hits, and what the actual memory and compute tradeoff looks like at the infrastructure level.

## Four-Property Self-Check

| Property | Status |
|---|---|
| Diagnostic | ✅ Names the specific mechanism gap: keys, values, bytes per token per layer, recomputation cost |
| Grounded in work | ✅ Tied to specific files and batch size in Week 10 system |
| Generalizable | ✅ Every FDE running batch inference with fixed context faces this |
| Resolvable in one explainer | ✅ One mechanism, one blog post closes it |