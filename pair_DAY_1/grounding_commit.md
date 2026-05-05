# Grounding Commit — Day 1

**Artifact edited:** Week 10 — Automated Lead Generation and Conversion System for Tenacious Consulting and Outsourcing  
**Specific location:** Batch inference prompt construction loop

## What Changed

Before this week's research, my Week 10 system constructed prompts by assembling context in whatever order was convenient — sometimes placing the lead-specific variable content before the fixed context files (`icp_definition.md`, `style_guide.md`, `bench_summary.json`), sometimes after.

After understanding the KV cache mechanism today, I restructured the prompt order so that all fixed context always comes first, followed by the variable lead data. The three fixed files are now loaded once, concatenated in a fixed order, and placed at the top of every prompt in the batch without modification.

## Why This Matters

The KV cache stores key-value matrices computed at each transformer layer for each token in the prefix. The cache key is a hash of the exact token sequence. If the fixed context appears after any variable content — or if its order shifts between calls — the cache sees a different token sequence on each call and recomputes the full prefix every time, eliminating the cost saving entirely.

With the restructured prompt order, the first call in a batch of 50 writes ~2,500 tokens to cache. Calls 2–50 read from cache, reducing prefill cost to approximately 10% of normal (Anthropic charges cache reads at 0.1× the standard input token rate). On a batch of 50 leads, this reduces prefill token cost by roughly 45× compared to the unstructured version.

## Commit Pointer

File: `lead_gen_system/batch_inference.py`  
Change: Moved `build_fixed_prefix()` call to execute once before the batch loop; moved `build_lead_context()` call inside the loop, appended after the fixed prefix.