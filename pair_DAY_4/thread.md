# Tweet Thread — Day 4

*Compressing my blog post: "Delta B Doesn't Know Why It Went Up: Detecting Length Bias in a Trained LLM Judge"*

---

**Tweet 1**

You trained a preference-tuned judge. Delta B went up. You conclude the judge improved.

But did it learn your rubric — or did it learn "longer output = preferred"?

A positive Delta B cannot tell you which one happened. Here's how to find out. 🧵

---

**Tweet 2**

The problem is structural, not accidental.

If your training pairs look like this:
- chosen = Claude Sonnet rewrite (~150 words)
- rejected = original agent output (~80 words)

Your judge saw 69 examples of: *longer → preferred*

The model optimizes for whatever predicts the label. If length is a more consistent signal than any single rubric feature, it learns length.

---

**Tweet 3**

Three tests, run in order:

**Test 1:** Measure chosen vs. rejected lengths in your training pairs.
→ Wilcoxon signed-rank test. Ratio > 1.3x and p < 0.05 = confound is real.

**Test 2:** On held-out data, compute Spearman ρ(token count, judge score).
→ ρ > 0.3, p < 0.05 = length is driving scores.

**Test 3:** Write 4 emails by hand — short compliant, long non-compliant, long compliant, short non-compliant. Score with your judge.
→ If the long non-compliant beats the short compliant, it's length bias.

---

**Tweet 4**

What Delta B *actually* tells you:

"On my 40 held-out tasks, the trained judge scored outputs 0.32 points higher on average than the raw backbone."

What it does NOT tell you:
- whether those outputs were longer or more rubric-compliant
- whether a word-count heuristic would produce similar lift
- whether the judge would score a short compliant email above a long non-compliant one

That last one is what matters in production.

---

**Tweet 5**

The operational risk:

In production, your judge is a rejection-sampling gate. It decides whether to send or regenerate.

A length-biased judge approves verbose rubric-violating emails. Rejects concise compliant ones.

Delta B looked fine during training. The failure mode was being baked in the whole time.

---

**Tweet 6**

The fix if the bias is real:

Truncate chosen outputs to within 10% of rejected length before retraining. Or length-match using paraphrasing. Don't downsample — if you only have 69 pairs, you can't afford to lose any.

Measure first. Retrain only if the correlation is real.

Full write-up with code: [blog post link]

Sources: Dubois et al. 2024 (Length-Controlled AlpacaEval) + Wang et al. 2023 (LLMs are not Fair Evaluators)
