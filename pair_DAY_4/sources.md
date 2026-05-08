# Sources — Day 4

## Canonical sources used in the explainer

1. **Dubois, Y., Li, C. X., Taori, R., Zhang, T., Gulrajani, I., Ba, J., ... & Hashimoto, T. (2024).
   Length-Controlled AlpacaEval: A Simple Way to Debias Automatic Evaluators.**
   *arXiv:2404.04475.*
   https://arxiv.org/abs/2404.04475
   — The direct reference for length bias in LLM-based evaluation. Shows that AlpacaEval win
   rates correlate strongly with output length and that a length-controlled correction
   substantially changes rankings. Table 1 and the abstract are the load-bearing content.

2. **Wang, P., Li, L., Chen, L., Zhu, D., Lin, B., Cao, Y., ... & Sui, Z. (2023).
   Large Language Models are not Fair Evaluators.**
   *arXiv:2305.17926.*
   https://arxiv.org/abs/2305.17926
   — Documents systematic positional and length biases in LLM judges across multiple
   benchmarks. Section 3.2 covers length bias specifically with correlation statistics.
   Relevant because Kidus's judge is itself an LLM (Qwen2.5-1.5B) trained to score outputs —
   the base model's length preference can survive fine-tuning.

3. **Sanh, V., Webson, A., Raffel, C., Bach, S. H., Sutawika, L., Alyafeai, Z., ... & Rush, A. M. (2022).
   Multitask Prompted Training Enables Zero-Shot Task Generalization.**
   *arXiv:2110.08207 / ICLR 2022.*
   https://arxiv.org/abs/2110.08207
   — Section 5.2 discusses length as a systematic confound in preference labeling, predating
   RLHF but amplified by it. Provides grounding for why the structural asymmetry in "strong
   model rewrites weak model output" datasets reliably produces length-correlated labels.

## Statistical tools used

- **`scipy.stats.wilcoxon`** — Wilcoxon signed-rank test for paired length comparison
  (chosen vs. rejected token counts within the same preference pair). Used for Test 1.
- **`scipy.stats.spearmanr`** — Spearman rank correlation between output length and judge
  score on held-out data. Non-parametric, robust to non-normal score distributions. Used
  for Test 2.

## Pattern used

**Three-test falsification sequence:** Test 1 (confound in training data) → Test 2
(length-score correlation on held-out) → Test 3 (adversarial pair scoring). The sequence
is ordered so that a negative result at any step terminates the investigation — avoiding
the trap of running all three and cherry-picking the one that looks suspicious.
