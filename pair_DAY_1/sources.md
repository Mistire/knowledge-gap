# Sources — Day 1

## Canonical Sources

1. **OpenRouter — Reasoning Tokens documentation**
   - URL: https://openrouter.ai/docs/guides/best-practices/reasoning-tokens
   - How it's load-bearing: confirms think tokens are billed as output tokens and appear in `completion_tokens`; documents the `include_reasoning` parameter for separating them in the response object

2. **QwenLM/Qwen3 GitHub Issue #1826 — Chat template breaks KV cache reuse when enable_thinking=false**
   - URL: https://github.com/QwenLM/Qwen3/issues/1826
   - How it's load-bearing: documents the specific KV cache reuse bug in multi-turn conversations when suppressing thinking mode; confirms the fix in vLLM 0.9.0

## Tool / Pattern Used

- **OpenRouter API call with `include_reasoning=True`** — made a live call to `qwen/qwen3-235b-a22b` with thinking enabled and inspected the `usage` object and `reasoning` field to verify that `completion_tokens` includes think tokens and that think content is accessible separately via `extra_body`

## Additional Reference

- **vLLM Qwen3.5 Usage Guide**: https://docs.vllm.ai/projects/recipes/en/latest/Qwen/Qwen3.5.html — documents `enable_thinking` parameter syntax for vLLM-based deployments including the `chat_template_kwargs` pattern
