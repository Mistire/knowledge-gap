# Grounding Commit — Day 1

**Asker:** Mistire Daniel
**Artifact edited:** `week-10/conversion-engine/baseline.md`

## What Changed and Why

Added a "Prefix Caching Analysis" section to `baseline.md` below the p95 task duration numbers. The section names that the 201s p95 cannot be fully explained without knowing whether OpenRouter was serving the system prompt from a KV cache hit or recomputing it from scratch on every call. It notes that the system prompt in `agent.py` is already structured correctly for caching (fixed, no dynamic content injection), but that cache hits depend on provider routing consistency — requests routed to different GPUs across calls will miss the cache regardless of prompt structure. It adds a verification step: check for `cache_read_input_tokens` in the API response or compare time-to-first-token across repeated identical calls. The section concludes that the p95 may be partially reducible through provider pinning on OpenRouter, not prompt changes.

## Pointer to Edit

File: `week-10/conversion-engine/baseline.md`
Section added after line 33 (p95 task duration row in the results table):

```markdown
### Prefix Caching Note

The p95 task duration of 201s includes prefill cost for the 83-line system prompt
on every LLM call. Whether this cost is eliminated by prefix caching depends on
OpenRouter routing consistency — if consecutive calls within a task hit the same
GPU, the system prompt KV blocks are reused and prefill is skipped for those tokens.
If routed to a different GPU, the cache misses and full prefill runs.

The system prompt in `agent/agent.py` is already cache-friendly: it is static,
injected first in every message list, and contains no dynamic content. The
cacheability condition is satisfied on the prompt side.

**Verification:** Check for `cache_read_input_tokens` in the OpenRouter API response,
or measure time-to-first-token on two consecutive identical calls. A measurably lower
TTFT on the second call indicates a cache hit.
```

*Grounding commit completed — 2026-05-05.*
