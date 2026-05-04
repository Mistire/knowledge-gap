# Tweet Thread — Qwen3 Think Tokens: You're Paying for What You Throw Away

---

**Tweet 1**
If you're using Qwen3 and stripping `<think>` blocks after generation — you're paying for tokens you never use. And they're slowing down your decoder too. Here's the mechanism. 🧵

---

**Tweet 2**
Qwen3's think tokens are not a separate reasoning pass. They're generated **in the same autoregressive sequence** as your response — token by token, before the actual answer.

```
<think>
...500 tokens of reasoning...
</think>
Here is your answer.   ← only 20 tokens you actually needed
```

You pay output token prices for all 520.

---

**Tweet 3**
It gets worse. By the time the model starts generating your actual answer, every think token is sitting in the **KV cache**. Every response token must attend over them.

500 think tokens = 500 extra attention positions per decode step. Your decoder slows down to generate the 20 tokens you actually wanted.

---

**Tweet 4**
How to verify: call the API with `include_reasoning=True` in `extra_body`. OpenRouter returns a separate `reasoning` field. Compare its token count to `completion_tokens`. The gap is what you're paying for and discarding.

```python
extra_body={"include_reasoning": True}
# response.usage.completion_tokens → includes think tokens
# response.choices[0].message.reasoning → the think block content
```

---

**Tweet 5**
The fix: suppress at inference time, don't strip after.

```python
extra_body={"chat_template_kwargs": {"enable_thinking": False}}
```

No think tokens generated → no billing, no KV cache inflation. Caveat: there's a known KV cache reuse bug with multi-turn conversations (fixed in vLLM 0.9.0 — check your provider's backend version).

---

**Tweet 6**
Full explainer with code, the KV cache mechanism, and the prefill/decode cost breakdown:
[link to blog post]

If you're running reasoning models in production multi-turn agents, this cost structure is invisible until you look for it.
