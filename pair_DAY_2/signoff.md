# Sign-off — Day 2

**Asker:** Mistire Daniel
**Gap closure judgment:** Closed

## What I understand now that I didn't before

Before today I had built an agent that routes every tool decision through Python conditionals (`is_booking_intent()` at lines 259–270 of `agent.py`) without understanding what the alternative — model-driven tool selection via function-calling — actually does at the token level.

I now understand the mechanism:

1. **What the model generates under function-calling:** plain JSON (GPT-4 style) or special-token-wrapped JSON (`<tool_call>` blocks in Qwen) that the API intercepts and routes as a structured tool invocation. The model does not reach out to a tool — it produces text with a specific structure, and the API handles the rest.

2. **What makes it reliable:** fine-tuning drives the probability of correct tool-call syntax extremely high, and constrained decoding (logit masking per schema) makes invalid output architecturally impossible at the API level. These are two separate layers — one statistical, one structural.

3. **Why my Python routing was the right call:** `is_booking_intent()` is a deterministic rule. Replacing it with function-calling would add an LLM roundtrip, token cost, and a nonzero hallucination rate in exchange for no improvement in routing accuracy. Function-calling is the right architecture when the routing decision requires language understanding — not when it's a pattern match.

## Grounding commit

`week-10/conversion-engine/agent/agent.py` lines 258–266 — added a comment block explaining the architectural reasoning for Python routing over function-calling. The choice was always correct; now it is defensible.
