# Day 2 Question — Function-Calling Token-Level Mechanics

## Final Sharpened Question

In my Week 10 lead gen agent, traces 19d13ac9 and 293b3bbb on task 92 show the agent looping for 682–687 seconds before hitting user_stop with reward 0.0. I cannot diagnose what happened inside that loop — whether the model was repeating tool calls, alternating between tools, or failing to emit a tool call at all — because I do not understand what is happening at the token level when a model "chooses" to call a tool versus generate text. Specifically: is the model outputting special tokens, a learned JSON schema, or something the API intercepts via constrained decoding? And which of those mechanisms, if broken or degraded, would produce a 682-second loop that the environment terminates rather than the agent? Without understanding the token-level mechanism, I cannot read my own traces.

## Artifact Pointer

**Week 10 system:** Automated Lead Generation and Conversion System for Tenacious Consulting and Outsourcing

Specifically: the agent loop in the conversion engine that calls tools like `get_company_info`, `get_job_posts`, and email composition functions. Traces 19d13ac9 and 293b3bbb on task 92 show the failure.

**What closing this gap changes:** I will be able to inspect a looping trace and identify — from the token-level mechanism — whether the failure was a malformed tool call JSON, a missing stop_reason, a max_tokens truncation mid-tool-call, or a genuine model planning failure where no tool had high enough probability to be selected.

## Four-Property Self-Check

| Property | Status |
|---|---|
| Diagnostic | ✅ Names specific traces, specific failure mode, specific mechanism gap |
| Grounded in work | ✅ Tied to real trace IDs and task number in Week 10 system |
| Generalizable | ✅ Every FDE building agents hits looping failures and needs to diagnose them |
| Resolvable in one explainer | ✅ One mechanism (token-level tool call) closes it |