# Why Your Agent Looped for 682 Seconds: Tool-Use Mechanics at the Token Level

*Week 12 · Day 2 · Written by Efrata Wolde for Zemzem Hibet*

---

## The Problem You're Looking At

Traces `19d13ac9` and `293b3bbb` on task 92 both show the same pattern: the agent runs for 682–687 seconds, gets terminated by `user_stop`, and returns `reward 0.0`. You can't diagnose what happened inside the loop because you don't understand what the model is actually doing when it "decides" to call a tool.

That's the gap this explainer closes. By the end you'll know exactly what mechanism produces a tool call, which of your three hypotheses is correct, and how to read your traces to identify which failure mode caused the loop.

---

## What Actually Happens When a Model Calls a Tool

The model has no separate decision module for tool calling. It does not "choose" to call a tool the way a human would choose. It predicts tokens — and a tool call is just a different kind of token sequence that the model has been fine-tuned to produce when the context warrants it.

Here's the sequence step by step:

**Step 1 — Tool schemas enter the context**
When you define tools in your agent, their schemas get injected into the model's context window — usually as a structured block in the system prompt. The model sees a JSON description of each tool: its name, what it does, and what parameters it takes. This is just text. The model reads it like everything else.

**Step 2 — The model predicts a structured output block**
When the model determines that a tool call is appropriate, it produces a structured JSON block instead of plain text. For Anthropic's API this looks something like:

```json
{
  "type": "tool_use",
  "id": "toolu_01abc",
  "name": "get_company_data",
  "input": {
    "company_name": "Acme Corp"
  }
}
```

This is the "tool call." The model itself does not execute anything. It just produces this block.

**Step 3 — The framework intercepts and executes**
Your agent framework (not the model) reads this block, routes it to the actual tool, runs the function, and gets a result back.

**Step 4 — The result re-enters the context**
The tool result gets injected back into the conversation as a `tool_result` block:

```json
{
  "type": "tool_result",
  "tool_use_id": "toolu_01abc",
  "content": "{ \"founded\": 2019, \"employees\": 42 }"
}
```

The model then reads this result and either calls another tool or generates a final text response.

This loop — predict tool call → execute → inject result → predict next action — is the entire mechanism. The model never "knows" it called a tool. It just sees more tokens in its context window.

---

## Your Three Hypotheses — Which Is Correct

**Special tokens** — Some open-source models (Llama, Mistral) use delimiter tokens like `<tool_call>` and `</tool_call>` to mark tool calls. The model was trained to produce these tokens at the right moments. This is correct for those models but not for Anthropic's API.

**Learned JSON schema** — This is what Anthropic uses. The model is fine-tuned on enormous quantities of tool-use examples until it reliably produces well-formed JSON matching the tool schema. There are no special tokens — the structure is entirely learned behaviour. This is the correct answer for your traces.

**Constrained decoding** — This is a technique used in frameworks like Outlines or Guidance where the API manipulates the model's output probabilities (logits) during generation to force output that matches a schema. This is a common assumption but it is not how Anthropic's or OpenAI's production APIs work. Those rely on fine-tuning, not logit manipulation. Your traces were not produced by constrained decoding.

**So: your traces show learned JSON schema production, not special tokens, not constrained decoding.**

---

## What Breaks and Produces a 682-Second Loop

There are three failure modes. Only one of them produces a 682-second loop terminated by the environment.

**Failure Mode 1 — Repeating the same tool call**
The model calls a tool, gets a result, and calls the exact same tool with the exact same arguments again. This happens when the tool result comes back in a format the model doesn't recognise as progress — so it tries again. The loop runs until the environment terminates it.

What it looks like in traces: identical `tool_use` blocks with identical `name` and `input` fields appearing multiple times in sequence.

**Failure Mode 2 — Alternating between tools without progress**
The model calls tool A, gets a result, calls tool B, gets a result, goes back to tool A. Neither result is sufficient to move the model toward a final answer so it keeps gathering information in a circle.

What it looks like in traces: alternating tool names with no final `text` response block between cycles.

**Failure Mode 3 — Failing to emit a tool call at all**
The model generates plain text instead of a structured tool call block — it "thinks out loud" without ever calling anything. This would show up as a stream of `text` blocks with no `tool_use` blocks.

**For a 682-second loop with `user_stop` and `reward 0.0`:** Failure Mode 3 is unlikely — a pure text generation loop would be faster and wouldn't burn 682 seconds. Your loop was actively executing something. That points to Failure Mode 1 or 2 — the agent was calling tools, getting results, and failing to move forward.

---

## How to Read Your Traces to Diagnose Which One

Open `19d13ac9` and `293b3bbb` and do these four checks in order:

**Check 1 — List all `mcp_tool_use` blocks in sequence**
Extract every block where `type == "mcp_tool_use"` and write down the `name` and `input` fields in order. If you see the same tool name with the same input appearing more than once → Failure Mode 1.

**Check 2 — Look for alternating tool names**
If the sequence goes `tool_A → tool_B → tool_A → tool_B` with no final text response → Failure Mode 2.

**Check 3 — Check `mcp_tool_result` blocks for errors**
Are results returning successfully or returning error messages? An error result that the model can't interpret as "stop trying" will cause it to retry indefinitely. Look at the `content` field of every `mcp_tool_result` block.

**Check 4 — Count `text` blocks in assistant turns**
If the model was generating text instead of tool calls, you'll see `type: text` blocks where `tool_use` blocks should appear. If there are no `tool_use` blocks at all → Failure Mode 3.

---

## What a Healthy Tool Loop Looks Like

For reference, a working single-step tool call produces this conversation structure:

```
user:      "Find company data for Acme Corp"
assistant: [tool_use: get_company_data, input: {company_name: "Acme Corp"}]
tool:      [tool_result: {founded: 2019, employees: 42}]
assistant: [text: "Acme Corp was founded in 2019 and has 42 employees."]
```

A broken loop looks like:

```
user:      "Find company data for Acme Corp"
assistant: [tool_use: get_company_data, input: {company_name: "Acme Corp"}]
tool:      [tool_result: ERROR - rate limit]
assistant: [tool_use: get_company_data, input: {company_name: "Acme Corp"}]
tool:      [tool_result: ERROR - rate limit]
assistant: [tool_use: get_company_data, input: {company_name: "Acme Corp"}]
... 682 seconds later ...
environment: user_stop
```

The model has no built-in retry limit. It will repeat until something external stops it — which is exactly what `user_stop` is.

---

## The One-Paragraph Answer

Tool calling works through fine-tuning, not special tokens or constrained decoding. The model is trained to produce structured JSON blocks that the framework intercepts and executes. The model never runs the tool — it just predicts what a tool call should look like. A 682-second loop with `user_stop` and `reward 0.0` almost certainly means the model was calling a tool, receiving a result it couldn't interpret as progress (likely an error or an unexpected format), and calling the same tool again — indefinitely. The environment terminated it because the model never emitted a final text response on its own.

---

*Sources: Anthropic Tool Use Documentation (docs.anthropic.com/tools) · ReAct: Synergizing Reasoning and Acting in Language Models (Yao et al., 2022)*