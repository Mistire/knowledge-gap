# Signoff — Day 1

**Asker:** Mistire Daniel
**Explainer written by:** Amir Ahmedin
**Gap:** Does OpenRouter re-compute my system prompt's KV cache on every call, or does prefix caching eliminate that cost — and what causes a cache miss?

## Gap Closure Judgment

## Status: Closed

## What I Understand Now That I Did Not Before

Before this explainer I knew prefix caching existed but could not explain what the cache key was or what broke it. I now understand that the cache key is the exact token sequence — one changed token invalidates the cache from that point forward — and that the most common miss in a managed API like OpenRouter is not a prompt change but provider routing: my request hitting a different GPU than the one that served the previous call. This means my system prompt in `agent.py` is already structured correctly (fixed, no dynamic injection), but whether I benefit from prefix caching depends on OpenRouter routing consistency, not my code. I can verify this by checking for `cache_read_input_tokens` in the API response or by measuring time-to-first-token on repeated identical calls. The p95 of 201s in my baseline may be partially reducible through provider pinning, not prompt changes.

---

*Signed off after evening call revision — 2026-05-05.*
