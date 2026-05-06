# Sources — Day 2

## Canonical Papers

1. **Willard & Louf (2023) — "Efficient Guided Generation for Large Language Models"**
   The foundational paper on grammar-constrained decoding. Describes the algorithm for computing valid token continuations from a formal grammar (including JSON schema) and applying logit masking at each generation step. This is the mechanism underlying OpenAI structured outputs, Outlines, and lm-format-enforcer.
   https://arxiv.org/abs/2307.09702

2. **OpenAI Structured Outputs documentation**
   Official explanation of how `response_format: json_schema` enforces schema validity at the token level. Describes the constraint layer and its relationship to function-calling.
   https://platform.openai.com/docs/guides/structured-outputs

## Tool / Pattern Used

**Outlines (dottxt-ai/outlines)**
Open-source library implementing grammar-constrained decoding for local models. Used to demonstrate the mechanism: generate JSON from a Qwen model with and without schema constraints, show the logit masking effect on malformed output rate.
https://github.com/dottxt-ai/outlines
