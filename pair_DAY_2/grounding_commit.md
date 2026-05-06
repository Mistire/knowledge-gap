# Grounding Commit — Day 2

## Artifact edited

`week-10/conversion-engine/agent/agent.py` — lines 258–266 (booking intent gate in `handle_email_reply`)

## What changed

Added a comment block above the `is_booking_intent()` routing gate explaining why Python routing was chosen over model-driven tool selection via function-calling.

## Why it changed

Before today, the comment at line 258 said "Check for booking intent before LLM call (saves cost)" — which is true but shallow. It describes what the code does, not why this architecture is defensible.

After researching function-calling at the token level, I now understand the actual trade-off: function-calling works by having the model generate structured JSON (or special-token-wrapped JSON for Qwen) that the API intercepts and routes as a tool invocation. That output is either constrained by the API (logit masking per schema) or statistically reliable due to fine-tuning — but it always requires an LLM call, and the model can still hallucinate tool names or arguments that don't match the schema.

`is_booking_intent()` is a deterministic rule. Replacing it with function-calling would introduce:
- One additional LLM roundtrip per turn
- Token cost for generating the tool-call output
- A nonzero probability of incorrect tool selection

In exchange for nothing — the deterministic rule is already 100% accurate for this routing decision.

The comment now captures this reasoning so the architectural choice is defensible on review.

## What I understand now that I didn't before

Function-calling is not a general upgrade over Python routing. It is the right architecture specifically when the routing decision requires language understanding — ambiguous intent, multi-field reasoning, edge cases no explicit rule handles cleanly. When the routing decision is a deterministic rule, Python is strictly better. I could not articulate this before today because I didn't know what the model was actually doing under function-calling.
