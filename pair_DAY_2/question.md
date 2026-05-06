# Day 2 Question — Agent and Tool-Use Internals

**Asker:** Mistire Daniel
**Topic:** Agent and tool-use internals — function-calling at the token level
**Date:** Day 2, Week 12

## The Question

My Week 10 outreach agent (`agent/agent.py`, lines 259–270) gates every booking decision through a Python function — `is_booking_intent(reply_text)` — before the LLM is ever consulted. The same pattern appears at lines 334–367 for SMS routing. The model never sees a list of available tools. It never generates a tool-call. Every routing decision is made by Python conditionals, not the model.

**When a model "chooses" a tool via function calling, what is it actually generating at the token level — and what does the API do to make that output structurally reliable, as opposed to my current approach where Python makes all routing decisions before the LLM runs?**

Specifically: what tokens (or token sequences) does the model produce when it decides to invoke a tool? Is that output constrained by the API (forced JSON schema, special tokens, logit biasing), or does reliability come from prompt engineering alone? And what would I have to change in `agent.py` to let the model drive tool selection instead of Python?

## Connection to Existing Work

Knowing this would let me revise:

- **`week-10/conversion-engine/agent/agent.py` (lines 259–270 and 334–367)** — I made a conscious choice to gate routing through Python keyword matching rather than exposing tools to the model. I cannot currently defend whether that was the right call because I don't understand what the alternative actually does at the mechanism level.
- **`week-10/conversion-engine/baseline.md`** — The p95 task duration (201s) includes latency from multiple sequential LLM and API calls. I don't know whether switching to function calling would add a roundtrip (tool-call token generation → API parse → tool execution → result injection) or save one (fewer Python-side checks before the LLM call). Either way, my latency breakdown has a hidden component I cannot currently explain.
- **`week-11/tenacious-bench/model_card.md`** — The model card describes the LoRA adapter as a "rejection-sampling layer in front of the outreach generator." If I move to function calling, the adapter would need to score tool-call outputs in addition to text outputs. I don't know whether the scoring rubric changes when the model output is structured JSON rather than prose.

## Why This Matters Beyond My Work

Any FDE building a production agent chooses — usually implicitly — between scaffolded routing (Python decides which tools run) and model-driven tool selection (the model generates tool invocations and the API executes them). Most practitioners make this choice without understanding the token-level mechanics, which means they can't reason about when each approach breaks. Understanding what the model actually generates and how the API constrains it determines whether tool-selection failures are debugging targets in the model or in the surrounding code.

## Four-Property Check

| Property | Status |
|---|---|
| Diagnostic | Names the specific gap: what token-level output the model generates for a tool call, and how the API enforces structure on that output |
| Grounded | Tied to `agent.py:259-270` (email booking gate) and `agent.py:334-367` (SMS booking gate) — both are concrete instances of the architectural choice under question |
| Generalizable | Applies to every agent that routes between tools — the scaffolding-vs-model-driven decision is live in every production agent system |
| Resolvable | A colleague can answer this in 800 words with one working function-calling example showing the raw API request/response and the token-level structure |
