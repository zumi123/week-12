# Sign-off — Day 1

**Asker:** Zemzem Hibet  
**Gap:** KV cache mechanism at the transformer level — what keys and values are, how many bytes per token per layer they occupy, and why recomputation is expensive enough that caching produces a measurable cost reduction.  
**Sign-off status:** Partially Closed

## What I Understand Now That I Did Not Before

Before today I could verify prefix caching from API metadata and state the conditions for a cache hit, but I was treating the KV cache as a black box — I knew it worked without knowing what it stored or why the storage mattered.

The explainer I received closed that gap. The KV cache stores the key and value matrices computed at every transformer layer for every token in the prefix. For a model with L layers, H attention heads, and a head dimension of d, each token occupies 2 × L × H × d × (bytes per element) of memory — roughly 1–2 MB per token for a large model at fp16. Recomputing these matrices from scratch on every call requires a full forward pass through all L layers for every prefix token, which scales as O(n²) with sequence length due to the attention computation. Reading from cache is a memory lookup — O(n). That is why the cost reduction is real and measurable, and why the byte-identical prefix requirement is not arbitrary: the cache key is a hash of the exact token sequence. One token difference = cache miss = full recomputation.

This changes how I think about my Week 10 system. The fixed prefix must be structurally first in every prompt, byte-identical across all 50 calls, and long enough to exceed the minimum cacheable threshold (1,024 tokens on Anthropic). These are not style choices — they are the mechanistic requirements for cache hits.