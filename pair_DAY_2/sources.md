# Sources — Day 2

## Canonical Sources

### 1. Anthropic Tool Use Documentation — How Tool Use Works
- **URL:** https://platform.claude.com/docs/en/agents-and-tools/tool-use/implement-tool-use
- **Why load-bearing:** Primary source for the `tool_use` content block structure, `stop_reason` values, the tool result contract, and the canonical agent loop pattern. Authoritative documentation on what the raw API response looks like when a tool call is made — the exact mechanism the explainer is built around.

### 2. Patil et al. (2023) — Gorilla: Large Language Model Connected with Massive APIs
- **URL:** https://arxiv.org/abs/2305.15334
- **Why load-bearing:** The foundational paper on training LLMs to call real-world APIs reliably. Demonstrates that tool description quality directly and measurably affects call accuracy — the empirical basis for the claim that how you write tool descriptions determines whether the model calls correctly. Also establishes that function calling is a learned behavior from training trajectories, not a rule-based system.

## Tool Used

### Anthropic Python SDK — `client.messages.create` with `tools` parameter
- **What it is:** Official Anthropic Python SDK for making API calls with tool schemas
- **How it was used:** Ran the canonical agent loop pattern against a test tool to verify `stop_reason` behavior, `tool_use` block structure, and `tool_result` return format. Confirmed that `stop_reason: "tool_use"` is the correct loop condition and that `is_error: true` in a `tool_result` block is a valid return for failed tool executions.
- **Install:** `pip install anthropic`