# What a Model Actually Generates When It "Calls a Tool" — and Why Prompt-Engineering JSON Is Not the Same Thing

*Written for Bethlehem. Published at Medium.*

---

Bethlehem built an ORPO judge on Qwen 2.5 that outputs a structured JSON verdict on every B2B sales email. Her `scoring_evaluator.py` parses that verdict with `json.loads()`. About 2% of the time, the JSON is malformed and the pipeline crashes. Her system prompt says "Respond with JSON only." She wants to know: if she switched to native function-calling, would that fix it — and what would be mechanically different?

The answer requires understanding what the model is actually doing in each case. They are not the same operation with different syntax. They are architecturally different at the token level.

---

## Prompt-Engineered JSON: The Model Writes Free Text That Looks Like JSON

When you write "Respond with JSON only" in a system prompt, you are not enforcing a structure. You are adjusting a probability distribution.

The model generates tokens one at a time, sampling from its full vocabulary at each step. "Respond with JSON only" shifts the probability mass toward JSON-shaped completions because the model was trained on data where that instruction precedes JSON output. But the probability of non-JSON tokens never reaches zero. At any position — a closing brace, a comma, a quote — the model can sample a token that violates the schema. When it does, `json.loads()` raises an exception.

The 2% malformed rate Bethlehem measured is the tail of this distribution. It is not a bug in her prompt. It is the correct behavior of a statistical model with no structural constraint on its output.

---

## Native Function-Calling: The API Applies Constrained Decoding

When you define tools in the API and the model invokes one, something structurally different happens. The output is not free text that happens to look like JSON. The generation process itself is constrained.

The mechanism is called **constrained decoding** or **grammar-guided sampling**. At each token position, the API computes which tokens are valid continuations of the JSON schema you defined. Tokens that would violate the schema — a missing brace, an unexpected key, a mismatched type — have their logits set to negative infinity before sampling. They are removed from the candidate set entirely. The model can only sample from tokens that keep the output valid.

This is not a post-processing step. It happens during generation, at every token. Invalid JSON is architecturally impossible — not just unlikely.

What the model generates under function-calling looks like this for GPT-4:

```json
{"name": "score_email", "arguments": {"decision": "SUPPRESS", "rule": "anti_offshore", "reason": "...", "score": 1.0}}
```

For Qwen 2.5, the model uses special tokens added to its vocabulary during fine-tuning:

```
<tool_call>
{"name": "score_email", "arguments": {"decision": "SUPPRESS", "rule": "anti_offshore", "reason": "...", "score": 1.0}}
</tool_call>
```

In both cases, the schema you defined determines which tokens are valid at each position. The model cannot deviate from it.

---

## The Distinction That Matters for Bethlehem's Judge

Bethlehem's judge is not choosing between tools. It always outputs a verdict. That use case does not need function-calling — it needs **structured outputs**, which apply the same constrained decoding mechanism to regular completions.

OpenAI's `response_format: {"type": "json_schema", "json_schema": {...}}` does this. Libraries like Outlines and lm-format-enforcer do it for open models. The mechanism is identical to function-calling's constraint layer: logit masking at each token position based on a JSON schema.

The table below makes the difference concrete:

| Approach | Token constraint | Malformed JSON rate |
| --- | --- | --- |
| Prompt engineering ("Respond with JSON only") | None — full vocabulary at every step | Nonzero (Bethlehem's 2%) |
| Native function-calling | Logit masking per schema at every step | Zero by construction |
| Structured outputs (`json_schema`) | Logit masking per schema at every step | Zero by construction |

For `scoring_evaluator.py`, the fix is structured outputs — not function-calling. Function-calling is for agents that choose which tool to invoke. Structured outputs are for judges that always return a fixed schema. Both use the same underlying constraint mechanism; they differ in when they apply it.

---

## Why Fine-Tuning Alone Is Not Enough

You might ask: Qwen 2.5 was fine-tuned extensively on JSON output. Shouldn't that be enough?

Fine-tuning makes the high-probability completion look like correct JSON. It does not make incorrect JSON impossible. The model's output distribution has support across the entire vocabulary at every token position. Fine-tuning concentrates probability on valid continuations but never fully removes probability from invalid ones. The tail never reaches zero.

Constrained decoding sets the tail to zero explicitly. That is the difference between "very likely valid" and "valid by construction." For a kill-switch condition at 1%, the difference matters.

---

## What Bethlehem Should Change

In `scoring_evaluator.py`, add a JSON schema matching the judge's output format and pass it as `response_format` when calling the model. The model's output will then always be a valid, parseable JSON object matching the schema. The `json.loads()` call will never raise again — not because the model got better at formatting, but because the generation process no longer allows malformed output.

The 2% rate goes to zero. Not because anything about the model changed. Because the API now enforces what the system prompt only suggested.

---

## The Short Version

Prompt-engineering JSON and using native function-calling or structured outputs are not the same operation. Prompt engineering biases the output distribution toward JSON. Constrained decoding removes non-JSON tokens from the candidate set at every step. One is a statistical preference. The other is a structural guarantee. For a judge that must always output a parseable verdict, the guarantee is the right choice.

---

*Sources in `sources.md`.*
