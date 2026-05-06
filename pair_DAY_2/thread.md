# Tweet Thread — Day 2

---

**Tweet 1**
"Respond with JSON only" in your system prompt is not a JSON guarantee. It's a probability nudge. The model can still sample a malformed token at any position. Here's what actually guarantees valid JSON — and why it's architecturally different. 🧵

---

**Tweet 2**
When you prompt-engineer JSON, the model generates tokens from its full vocabulary at every step. A closing brace, a comma, a quote — any of these can go wrong. "JSON only" shifts the distribution toward valid output. It never makes invalid output impossible. That's why you still see 2% malformed rates.

---

**Tweet 3**
Native function-calling and structured outputs use a different mechanism: constrained decoding. At every token position, the API computes which tokens are valid continuations of your JSON schema. Invalid tokens get their logits set to -∞. They are removed from the candidate set entirely. Malformed JSON becomes architecturally impossible — not just unlikely.

```
Prompt engineering:  P(malformed) → low but nonzero
Constrained decoding: P(malformed) = 0 by construction
```

---

**Tweet 4**
For GPT-4, function-calling outputs plain JSON that the API intercepts and routes through `tool_calls`. For Qwen 2.5, the model generates special vocabulary tokens `<tool_call>` ... `</tool_call>` added during fine-tuning. Different syntax. Same constraint layer underneath.

---

**Tweet 5**
Important distinction: function-calling is for agents choosing between tools. Structured outputs (`response_format: json_schema`) is for judges that always return a fixed schema. Both apply the same logit-masking mechanism. If your model always returns a verdict — use structured outputs, not function-calling.

---

**Tweet 6**
Full explainer — what the model generates at the token level, why fine-tuning alone isn't enough, and what to change in `scoring_evaluator.py` to get the malformed rate to zero:

[Medium link — TBD]
