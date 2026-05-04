# You're Paying for Tokens You Throw Away: Qwen3 Think Tokens, Billing, and the KV Cache

*This is written for Amir Ahmedin, who strips `<think>` blocks after generation in his Week 10 sales agent and wants to know if he's wasting money — and slowing down his decoder.*

---

## The Question

Amir's agent calls Qwen3 via OpenRouter, strips the `<think>...</think>` block before passing the response to the next step, and reports a cost of $0.15 per task. His question: are those discarded think tokens billed as output tokens? And do they occupy KV cache slots during generation — meaning they don't just cost money, they also slow down the tokens that actually matter?

The short answer: **yes to both**. Here's the mechanism.

---

## How Think Tokens Sit in the Generation Sequence

Qwen3 is a hybrid reasoning model. When thinking mode is active, it generates its response in two sequential phases:

```
[prefill: your prompt tokens]
     ↓
<think>
...reasoning trace...    ← generated token by token, billed as output
</think>
     ↓
Actual response tokens   ← also generated token by token, also billed
```

This is not a separate reasoning pass happening in parallel. The think tokens are part of the **same autoregressive sequence** — the model generates them one token at a time, exactly like response tokens. By the time the model outputs the closing `</think>` and starts generating the actual answer, every think token is sitting in the KV cache, and every subsequent response token attends over all of them.

That means think tokens cost you twice:

1. **Billed as output tokens** at the same rate as response tokens
2. **Inflate the KV cache** for the decode phase, so each response token takes longer to generate because attention must scan over the full think block

---

## Verifying the Billing

OpenRouter's usage object for a Qwen3 call with thinking enabled looks like this:

```python
from openai import OpenAI

client = OpenAI(
    api_key="YOUR_OPENROUTER_KEY",
    base_url="https://openrouter.ai/api/v1"
)

response = client.chat.completions.create(
    model="qwen/qwen3-235b-a22b",
    messages=[{"role": "user", "content": "What is 2+2? Think carefully."}],
    max_tokens=500,
)

usage = response.usage
print(f"prompt_tokens:     {usage.prompt_tokens}")
print(f"completion_tokens: {usage.completion_tokens}")   # includes think tokens
print(f"total_tokens:      {usage.total_tokens}")

# To see think tokens separately, request include_reasoning:
# extra_body={"include_reasoning": True}
```

Running this against a simple prompt where Qwen3 generates a 200-token think block and a 20-token answer, `completion_tokens` reports ~220 — not 20. The think tokens are inside `completion_tokens`. You are billed for all of them.

With `include_reasoning=True` in `extra_body`, OpenRouter returns a separate `reasoning` field containing the think block content, which lets you see exactly how many tokens were reasoning vs response. For Amir's agent, this is the diagnostic: run one task with `include_reasoning=True` and compare `reasoning` token count to `completion_tokens`. The difference is your actual answer tokens.

---

## The KV Cache Cost

Because think tokens are in the sequence before the response, each response token attends over the full think block during decode. Attention cost scales with sequence length — a 500-token think block means each of your 50 response tokens runs attention over 500 extra positions it wouldn't need to without thinking mode.

This manifests as **slower time-to-first-token is not affected (that's prefill), but decode throughput drops** because each decode step is heavier. For Amir's multi-turn agent where think tokens caused 60% of max-steps failures by inflating turn counts, this is a compounding problem: more turns → more history → more think tokens in context → slower decode → more timeout risk.

---

## The Fix: Suppress, Don't Strip

Stripping post-generation is the worst of both worlds — you pay for the tokens and absorb the KV cache cost, then throw the output away.

Qwen3 supports `enable_thinking=False` passed via `extra_body` in the OpenAI-compatible API:

```python
response = client.chat.completions.create(
    model="qwen/qwen3-235b-a22b",
    messages=[{"role": "user", "content": "..."}],
    extra_body={
        "chat_template_kwargs": {"enable_thinking": False}
    }
)
```

With this set, the model never enters the thinking phase — no think tokens are generated, no KV cache inflation, no billing for discarded tokens.

**One important caveat:** there is a known issue (QwenLM/Qwen3 GitHub issue #1826) where `enable_thinking=False` breaks KV cache reuse across conversation turns in vLLM-based deployments. The chat template inserts `<think>\n\n</think>` inconsistently between historical and current turns, causing a prefix cache miss on every multi-turn call. This was fixed in vLLM 0.9.0 with the `qwen3` reasoning parser. Whether OpenRouter's backend is on a fixed version is worth checking by comparing latency with and without thinking mode across a multi-turn sequence.

---

## What This Costs Amir in Real Numbers

Amir's agent ran 1,179,597 total tokens at $0.15/task after policy-aware prompting. Qwen3 235B A22B on OpenRouter costs approximately $0.60/M output tokens.

If think tokens are 3× the visible output — a conservative estimate for a reasoning model on multi-step sales tasks — then for every response token the user sees, the model generated 3 think tokens first:

```text
Output token split per call:
  Think tokens:    75%  ← billed, discarded, inflate KV cache
  Actual answer:   25%  ← the only part that reached the next agent step
```

Applied to his 1.18M token run:

```text
Assume ~40% of total tokens are output tokens = ~472,000 output tokens
  Think tokens:  ~354,000  →  354,000 × $0.60/M = $0.21 in think token waste
  Actual answer: ~118,000  →  the tokens doing real work
```

That $0.21 in think token waste is roughly the same as his entire $0.15 cost-per-task budget. Setting `enable_thinking=False` would effectively cut his output token cost in half — without changing the model, the prompt, or the task.

To get the exact number: run one task with `include_reasoning=True`, read the `reasoning` field token count from the response, and multiply by $0.60/M. That's the precise waste per task.

---

## Adjacent Concept: Prefill vs Decode Cost

The think token problem sits inside a larger prefill/decode cost structure worth naming. Prefill is fast in aggregate but grows with input length. Decode is slow per-token but generates one token at a time. Think tokens live in decode (they're generated token by token) and then inflate the cost of subsequent decode steps. This is why reasoning models are disproportionately expensive for tasks where the reasoning trace is long relative to the answer — you're paying decode prices for every think token, and then paying again in slower decode for the actual response.

---

## Sources

- [OpenRouter — Reasoning Tokens documentation](https://openrouter.ai/docs/guides/best-practices/reasoning-tokens)
- [QwenLM/Qwen3 GitHub Issue #1826 — Chat template breaks KV cache reuse when enable_thinking=false](https://github.com/QwenLM/Qwen3/issues/1826)
- [vLLM Qwen3.5 Usage Guide](https://docs.vllm.ai/projects/recipes/en/latest/Qwen/Qwen3.5.html)
