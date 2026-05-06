# Morning Call Summary — Day 2

**Pair:** Mistire Daniel & Bethlehem Abiy
**Topic:** Agent and tool-use internals
**Date:** Day 2, Week 12

## What was ambiguous in Mistire's original draft

The original question asked about "structural reliability" without defining it. Bethlehem pushed back: was the question about the token-level mechanism (what the model generates), or about whether to refactor `agent.py` to use function-calling instead of Python routing? The two are different questions requiring different explainers.

**How it was sharpened:** Mistire clarified the question is strictly the mechanism — what the model generates at the token level and how the API enforces valid structure. The architectural decision (whether to refactor) is the grounding commit that follows from understanding the mechanism, not part of the question itself.

## What was ambiguous in Bethlehem's original draft

Bethlehem's original question conflated function-calling (model chooses between tools) with constrained decoding (model output forced to conform to a JSON schema). Her judge always outputs a verdict — it is not choosing between tools — so function-calling is the wrong mechanism for her use case.

**How it was sharpened:** Bethlehem clarified that what she built is prompt-engineered JSON output, and what she needs is structured outputs / constrained decoding. Her final question focuses on the mechanism difference between these two: what happens at the token level in each case, and whether constrained decoding guarantees valid JSON by construction.

## Final committed questions

**Mistire:** When a model "chooses" a tool via function calling, what is it actually generating at the token level — and what does the API do to ensure the output is always valid, parseable tool syntax rather than free text?

**Bethlehem:** At the token level, what is the mechanism that makes native function-calling structurally reliable — and can that same mechanism (constrained decoding, grammar sampling, or schema enforcement) be applied to my judge's JSON output without switching to the OpenAI tools API?
