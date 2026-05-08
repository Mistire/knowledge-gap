# Delta B Doesn't Know Why It Went Up: Detecting Length Bias in a Trained LLM Judge

Your question points to a real blind spot in preference-training evaluation. Your
`ablation_results.json` shows Delta B = +0.3204 — the trained judge scores held-out
outputs significantly higher than the raw backbone. But your training pairs have a structural
asymmetry baked in: chosen outputs are Claude Sonnet rewrites, rejected outputs are original
agent outputs. Claude Sonnet writes longer emails than a Week 10 outreach agent. If the
chosen outputs are systematically longer, the judge had 69 examples that all said the same
thing: *longer output → preferred*. Delta B cannot tell you whether it learned the rubric
or learned that rule. This explainer is about how to find out.

---

## The Load-Bearing Mechanism: What Length Bias Actually Is

When you train a preference model on pairs where the chosen output is consistently longer,
the model learns a spurious correlation. The rubric is the intended signal. Length is a
confound that correlates with the rubric in your training data but does not cause the
quality difference. The model cannot distinguish between the two — it optimizes for whatever
predicts the preference label most reliably. If length is a stronger or more consistent
signal than any individual rubric feature, the model learns length.

This is a form of reward hacking at the data level, not the model level. The model is doing
exactly what it was trained to do. The problem is in the pairs, not the optimizer.

A positive Delta B is consistent with three different situations:

1. The judge learned the rubric and scores rubric-compliant outputs higher.
2. The judge learned length and scores longer outputs higher, and held-out "better" outputs
   happen to also be longer.
3. Both — length and rubric compliance are both drivers, entangled.

Without measuring the length distribution in the training pairs and the length-score
correlation on held-out data, you cannot tell these apart.

---

## Show It: Three Tests in Order

Run these three tests in order. Stop if an earlier one clears the concern.

**Test 1: Is the confound present in the training data?**

```python
import json
import numpy as np
from scipy import stats

# Load your 69 preference pairs — adjust path as needed
with open('preference_pairs.json', 'r') as f:
    pairs = json.load(f)

# Use word count as a proxy for token count (close enough for this check)
chosen_lengths = [len(p['chosen'].split()) for p in pairs]
rejected_lengths = [len(p['rejected'].split()) for p in pairs]

print(f"Chosen   — mean: {np.mean(chosen_lengths):.1f}, median: {np.median(chosen_lengths):.1f}")
print(f"Rejected — mean: {np.mean(rejected_lengths):.1f}, median: {np.median(rejected_lengths):.1f}")
print(f"Ratio (chosen/rejected): {np.mean(chosen_lengths)/np.mean(rejected_lengths):.2f}x")

# Wilcoxon signed-rank test (paired — same task, two versions)
stat, p = stats.wilcoxon(chosen_lengths, rejected_lengths)
print(f"Wilcoxon signed-rank p = {p:.4f}")
```

**Interpretation:** If the ratio is above ~1.3x and p < 0.05, the confound is present in
the training data and you must run Tests 2 and 3. If the lengths are roughly equal, stop
here — length bias is not the mechanism.

---

**Test 2: Does length predict judge score on held-out data?**

```python
from scipy.stats import spearmanr

# For each held-out output, collect: token count and judge score
# Adjust to match your ablation data structure
output_lengths = [len(output.split()) for output in held_out_outputs]
judge_scores = [scores[i] for i in range(len(held_out_outputs))]

rho, p = spearmanr(output_lengths, judge_scores)
print(f"Spearman ρ(length, score) = {rho:.3f}, p = {p:.4f}")
```

**Interpretation:** ρ > 0.3 and p < 0.05 means length is a real driver of scores. ρ < 0.1
and p > 0.1 means scores are largely independent of length — rubric is the dominant signal.
Values in between are ambiguous and warrant Test 3.

---

**Test 3: The adversarial pair test**

Construct four minimal pairs manually — do not use the trained judge to generate these:

| Output | Rubric | Length |
|--------|--------|--------|
| A | Compliant | Short (~60 words) |
| B | Non-compliant | Long (~180 words) |
| C | Compliant | Long (~180 words) |
| D | Non-compliant | Short (~60 words) |

Score all four with the trained judge. A rubric-learning judge should rank A > B and C > D
regardless of length. A length-biased judge will rank B > A and C > D — length wins over
rubric in the first pair. If B scores higher than A, the judge is length-biased.

This test doesn't require code. It requires 20 minutes and four hand-written emails.

---

## What Delta B Alone Fails to Tell You

Delta B = +0.3204 means: on your 40 held-out tasks, the trained judge assigned scores that
were on average 0.32 points higher than the raw backbone's scores. It does not tell you:

- Whether the held-out outputs the trained judge scored higher were longer, more
  rubric-compliant, or both.
- Whether a judge that simply counted words would produce a similar Delta B on the same data.
- Whether the judge would score a short, rubric-compliant output above a long,
  rubric-violating one — which is the decision that matters in production.

The last point is the operational risk. In production, the judge is a rejection-sampling
layer: it decides whether to send an email or regenerate it. If the judge rewards length,
it will approve verbose rubric-violating emails and reject concise rubric-compliant ones.
Delta B would have looked the same during training while this failure mode was being baked in.

---

## Concrete Fix for `methodology_rationale.md`

Add these two sentences to your preference pair construction section:

> "Chosen outputs (Claude Sonnet rewrites) and rejected outputs (original agent outputs) were
> not length-matched before training. Mean word count: chosen = X words, rejected = Y words
> (ratio = Z). Spearman ρ between output length and judge score on held-out data = [value].
> Length bias was [ruled out / not ruled out] based on Test 1–2 results."

If the lengths are imbalanced and you need to retrain, the fix is length-matched pairs:
truncate chosen outputs to within 10% of the rejected length, or upsample rejected outputs
to match chosen lengths using a paraphrasing step. Do not downsample — you only have 69 pairs.

---

## Why This Generalizes

Any FDE who generates preference pairs by taking rewrites from a more capable model and
pairing them against original outputs will hit this problem. More capable models write
differently — usually more thoroughly, which usually means longer. The length correlation
is structural, not accidental, and it appears in almost every "strong model rewrites weak
model output" dataset construction pattern. The fix is always the same: measure first,
retrain only if the correlation is real.

---

## Pointers

- Sanh, V. et al. (2022). "T0: Multitask Prompted Training." ICLR 2022. Section 5.2 discusses
  length as a systematic confound in evaluation and preference labeling — the problem predates
  RLHF but is amplified by it.
- Dubois, Y. et al. (2024). "Length-Controlled AlpacaEval." arXiv 2404.04475. The direct
  reference: shows that AlpacaEval win rates correlate strongly with output length and
  proposes a length-controlled correction. Read the abstract and Table 1 — the core finding
  is one page.
- Wang, P. et al. (2023). "Large Language Models are not Fair Evaluators." arXiv 2305.17926.
  Shows that LLM judges have systematic positional and length biases. Section 3.2 covers
  length bias specifically with correlation statistics.
- Tool: `scipy.stats.spearmanr` for the correlation test, `scipy.stats.wilcoxon` for the
  paired length comparison. Both are in the standard scientific Python stack.