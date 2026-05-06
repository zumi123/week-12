# Tweet Thread — Day 2

---

**Tweet 1**
Your LLM agent "decides" to call a tool. But what is actually happening at the token level? Is it special tokens? Constrained decoding? A learned JSON schema? And what breaks when it loops for 682 seconds?

Here's the mechanism. 🧵

---

**Tweet 2**
When you pass a `tools` array to the API, two things happen:

1. Tool schemas are serialized into the prompt as tokens — the model *reads* your tool descriptions the same way it reads your system prompt

2. The model is trained to emit a `tool_use` content block when it decides to call a tool

The raw response looks like this:

```json
{
  "type": "tool_use",
  "id": "toolu_01XFD...",
  "name": "hubspot_create_contact",
  "input": { "email": "sarah@tenacious.com" },
  "stop_reason": "tool_use"
}
```

---

**Tweet 3**
The model does NOT execute the tool.

It emits a structured request. Your code runs the tool. You return a `tool_result`. The model continues.

This contract is absolute. Your agent loop should be keyed on ONE field: `stop_reason`.

→ `"tool_use"` = execute tool, return result, continue
→ `"end_turn"` = done, render response
→ `"max_tokens"` = truncated, handle explicitly — NOT silently retry

---

**Tweet 4**
Why does your agent loop for 682 seconds?

Three token-level causes:

1. **Truncated tool call** — hit max_tokens mid-JSON, loop retries forever
2. **Missing tool_result** — tool threw an exception, you never returned the result, model calls the same tool again
3. **Planning loop** — model completed step A, got result, has no mechanism to know it already did step A

All three are scaffolding bugs, not model bugs.

---

**Tweet 5**
The fix for all three:

```python
except Exception as e:
    tool_results.append({
        "type": "tool_result",
        "tool_use_id": block.id,
        "content": str(e),
        "is_error": True  # ← always return a result
    })
```

And add completed-step tracking to your system prompt:
`"Already completed: [get_company_info for sarah@tenacious.com]"`

The model cannot track state it cannot see.

---

**Tweet 6**
Full explainer: the token-level mechanism, the raw API response shape, the canonical loop pattern, and the three failure modes that cause agent loops — with runnable code.

[BLOG POST LINK]

Sources:
→ Anthropic tool use docs: platform.claude.com/docs/en/agents-and-tools/tool-use
→ Gorilla paper (Patil et al. 2023): arxiv.org/abs/2305.15334