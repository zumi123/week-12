# What Actually Happens When an LLM "Decides" to Call a Tool

*Written for a colleague whose conversion engine passes HubSpot and Resend tool schemas to an LLM and needs to understand what happens at the token level when the model chooses to call a tool versus respond in plain text.*

---

## The Question That Matters

Your conversion engine defines tools — `hubspot_create_contact`, `resend_send_email`, and others. You pass their schemas to the model. The model "decides" to call one. But what is the model actually doing? Is it reasoning about the tool, then outputting JSON? Is something constraining its output? And what determines whether it calls a tool at all versus just responding in text?

These are not academic questions. If you do not understand the mechanism, you cannot diagnose why your agent sometimes loops, skips a tool it should have called, or emits a malformed tool call that silently fails.

---

## The Load-Bearing Mechanism

When you send a request with a `tools` array to the Anthropic API, two things happen that do not happen in a plain text request.

**First, the tool schemas are serialized into the prompt.** The model does not receive tool schemas as a separate data structure — they are injected into the context as tokens, typically in a system-turn block before your messages. Your `hubspot_create_contact` schema, its description, and its input parameters all become part of the token sequence the model reads. This is why tool descriptions matter: the model reads them the same way it reads your system prompt.

**Second, the model is trained to emit a specific output structure when it decides to use a tool.** Rather than generating free text, it outputs a `tool_use` content block — a structured JSON object containing `type`, `id`, `name`, and `input` fields. Here is what the raw API response looks like when Claude calls a tool:

```json
{
  "role": "assistant",
  "content": [
    {
      "type": "tool_use",
      "id": "toolu_01XFDUDYJgAACTJgxvc5b99d",
      "name": "hubspot_create_contact",
      "input": {
        "email": "sarah@tenacious.com",
        "firstname": "Sarah",
        "company": "Tenacious Consulting"
      }
    }
  ],
  "stop_reason": "tool_use"
}
```

The critical field is `stop_reason: "tool_use"`. This is the signal your agent loop checks. When `stop_reason` is `"tool_use"`, your code is expected to execute the tool and return a `tool_result` block. When `stop_reason` is `"end_turn"`, the model is done and you render the response. Your loop should be keyed on this field — nothing else.

**The model does not execute the tool.** It emits a structured request. Your code runs the tool. This contract is absolute.

---

## How the Model Decides: Tool Call vs Plain Text

The model's decision to call a tool versus generate text is a learned behavior, not a rule. During training, the model was exposed to thousands of trajectories where calling a tool was the correct next action — and thousands where it was not. The model learned to recognize the pattern: when a user request maps to a capability described in the tools array, emit a `tool_use` block; when it does not, generate text.

Two practical factors determine whether the model calls a tool on any given turn:

**Tool description quality.** The model matches the user's intent against tool descriptions using the same mechanism it uses for everything else: next-token prediction over the serialized tool schemas. A description like `"Creates a contact in HubSpot"` gives the model less signal than `"Creates a new contact record in HubSpot CRM. Use this when you have a qualified lead's email address and need to add them to the pipeline before sending outreach."` The second description names the condition under which the tool should be called — the first does not.

**The `tool_choice` parameter.** You can force tool use with `tool_choice: {type: "any"}` (must call some tool) or `tool_choice: {type: "tool", name: "hubspot_create_contact"}` (must call this specific tool). The default `auto` lets the model decide. For production agents where tool calls are mandatory at certain steps, `tool_choice` is the reliable enforcement mechanism — not prompt instructions alone.

---

## The Failure Mode That Causes Looping

Here is the mechanism behind agent loops. Your agent loop runs while `stop_reason == "tool_use"`. If the model keeps emitting tool calls without ever emitting `stop_reason: "end_turn"`, the loop runs indefinitely.

Three token-level causes:

**Truncated tool calls.** If the model hits `max_tokens` mid-tool-call, the `tool_use` block is incomplete. The API returns `stop_reason: "max_tokens"` instead of `"tool_use"`, but if your loop does not check for this, it may retry indefinitely. Fix: always check for `stop_reason: "max_tokens"` and handle it explicitly.

**Tool result not returned.** After calling a tool, the model expects a `tool_result` block in the next user message. If your code fails to return the result — due to an exception, a timeout calling HubSpot, or a silent error — the model sees an incomplete conversation and may call the same tool again. Fix: wrap every tool execution in try/catch and always return a `tool_result`, even on failure, with `is_error: true`.

**Planning loop.** The model calls tool A, gets a result, calls tool B, gets a result, then calls tool A again because its planning logic has not changed state. This is a scaffolding problem: the model lacks a mechanism to recognize it has already completed a step. Fix: add a step-tracking field to your agent state and include it in the system prompt: `"Already completed: [get_company_info for sarah@tenacious.com]"`.

```python
# The canonical loop shape — keyed on stop_reason
while True:
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=1024,
        tools=tools,
        messages=messages
    )
    
    messages.append({"role": "assistant", "content": response.content})
    
    # Always check stop_reason first
    if response.stop_reason == "end_turn":
        break
    elif response.stop_reason == "max_tokens":
        # Handle truncation — do not silently retry
        raise RuntimeError("Response truncated mid-tool-call")
    elif response.stop_reason == "tool_use":
        tool_results = []
        for block in response.content:
            if block.type == "tool_use":
                try:
                    result = execute_tool(block.name, block.input)
                    tool_results.append({
                        "type": "tool_result",
                        "tool_use_id": block.id,
                        "content": result
                    })
                except Exception as e:
                    # Always return a result — even on failure
                    tool_results.append({
                        "type": "tool_result",
                        "tool_use_id": block.id,
                        "content": str(e),
                        "is_error": True
                    })
        messages.append({"role": "user", "content": tool_results})
```

---

## The Adjacent Concept Worth Knowing

Constrained decoding (logit biasing) is sometimes mentioned in the context of tool calling — the idea that invalid tokens are masked to guarantee valid JSON output. This exists for structured outputs (`output_config.format`), but standard tool calling in Claude does not use hard logit masking. The model generates valid tool call JSON through training, not through token-level constraints. This distinction matters: it means a model under distribution shift (unusual tool names, atypical input schemas) can still produce malformed tool calls. Strict mode (`strict: true` in the tools array) adds schema validation at the API level and will reject malformed calls before they reach your code.

---

## What to Change in Your Conversion Engine

Check your agent loop for three things:
1. Is `stop_reason` checked explicitly for all values (`end_turn`, `tool_use`, `max_tokens`)?
2. Does every tool execution path — including exceptions — return a `tool_result` block?
3. Does your system prompt include any mechanism for the model to track what steps have already been completed?

If any answer is no, you have a latent looping bug.

---

## Sources

- **Anthropic tool use documentation** — canonical description of the `tool_use` content block, `stop_reason` values, and the tool result contract: https://platform.claude.com/docs/en/agents-and-tools/tool-use/implement-tool-use
- **Patil et al. (2023), Gorilla: Large Language Model Connected with Massive APIs** — the foundational paper on training LLMs to call APIs reliably; explains why tool description quality directly affects call accuracy: https://arxiv.org/abs/2305.15334
- **Tool used:** Anthropic Python SDK `client.messages.create` with `tools` parameter — ran the canonical loop pattern against a test tool to verify `stop_reason` behavior and `tool_use` block structure.