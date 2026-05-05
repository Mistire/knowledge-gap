# Day 1 Question — Inference-Time Mechanics

**Asker:** Mistire Daniel
**Topic:** Inference-time mechanics — KV cache and prefix caching
**Date:** Day 1, Week 12

## The Question

My Week 10 outreach agent (`agent/agent.py`, line 51) sends the same ~83-line `SYSTEM_PROMPT` on every LLM call through OpenRouter. The agent is multi-turn — each conversation turn re-sends the full message history including this fixed system prompt. I ran 3 concurrent tasks across 5 trials (150 task-trials total) and reported p95 task duration of 201s in my baseline, but I never decomposed where that time goes.

**Does OpenRouter's prefix caching eliminate the prefill cost of my repeated system prompt across turns, and how would I verify whether it's working?**

If the system prompt tokens are re-computed on every call, then every turn in every multi-turn task is paying full prefill cost for ~83 tokens of context that never changes. If prefix caching is active and working, those tokens are served from cache after the first call. Either way, my p95 of 201s and my cost-per-task numbers in `baseline.md` have a hidden component I cannot currently explain or defend.

## Connection to Existing Work

Knowing this would let me revise:

- **`week-10/conversion-engine/baseline.md`** — The p95 task duration (201s) and p50 (85.9s) are reported with no breakdown of prefill vs decode cost. I would add a section decomposing how much of each call's latency is the system prompt prefill and whether prefix caching was active during my runs.
- **`week-10/conversion-engine/agent/agent.py` (line 51)** — If prefix caching requires a structurally stable prefix (system prompt always first, unchanged), I need to verify my message construction actually satisfies that. If not, a small restructure would eliminate repeated prefill cost.

## Why This Matters Beyond My Work

Any FDE building a multi-turn agent with a long system prompt through a managed API (OpenRouter, Together, Anthropic) faces the same invisible cost structure. Most practitioners report total latency without knowing how much is avoidable prefix recomputation. Understanding exactly what constitutes a cache key — and what breaks it — determines whether production agents can be made cheaper and faster without changing a single line of the model or prompt content.

## Four-Property Check

| Property | Status |
|---|---|
| Diagnostic | Names a specific mechanism: cache key structure and miss conditions |
| Grounded | Tied to `SYSTEM_PROMPT` in `agent.py:51` and the 201s p95 in `baseline.md` |
| Generalizable | Applies to every multi-turn agent with a fixed system prompt on any managed API |
| Resolvable | A colleague can answer this in 800 words with one API experiment and the provider docs |
