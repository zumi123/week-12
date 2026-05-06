# Sign-off — Day 2

**Gap:** Token-level function-calling mechanism — what the model outputs when it calls a tool, how it decides to call vs respond in text, and which failure modes produce agent loops.
**Sign-off status:** ✅ Closed

## What I Understand Now That I Did Not Before

Before today I knew my agent was looping but had no framework for diagnosing why. I was treating the tool call as a black box — I could see the symptom (682-second loop, user_stop) but not the mechanism.

The explainer I received closed three gaps at once. First: the model emits a `tool_use` content block with `stop_reason: "tool_use"` — this is the signal my loop should key on, nothing else. Second: the model never executes the tool — it emits a request and waits for a `tool_result`. If my code fails to return a result for any reason, the model sees an incomplete conversation and may call the same tool again. Third: `max_tokens` truncation mid-tool-call produces `stop_reason: "max_tokens"`, not `"tool_use"` — if my loop does not check for this explicitly, it will retry indefinitely.

Looking at traces 19d13ac9 and 293b3bbb now, the 682-second duration suggests either a missing tool_result return on exception (the tool call to a slow external API timed out and the exception was swallowed) or a planning loop where the model had no mechanism to track completed steps. I can now instrument the agent to distinguish these.