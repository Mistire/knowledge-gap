# Morning Call Summary — Day 4

**Partners:** Mistire Daniel (asker) + Kidus Tewodros (explainer)
**Topic:** Evaluation and statistics

## What was ambiguous in the original draft

The original draft question asked broadly how to detect length bias in a judge and what
Delta B fails to tell you. The partner pushed back on two things:

First, the question pointed to `ablation_results.json` (held-out Delta B) as the grounding
artifact, but the length-bias problem lives in the training pairs — not in the held-out
evaluation. The held-out data can confirm the bias is active, but the confound itself is
structural in the 69 pairs. Pointing to the wrong artifact made the question harder to
answer concretely.

Second, "detect whether scores are driven by length" was underspecified. Detect how? With
what test? On what data? The question needed to name either the training-data check (are
chosen outputs longer?) or the held-out check (does length predict score?) — not merge
them into a single vague ask.

## How the question was sharpened

The question was split into its two distinct components: (1) is the confound present in
the training data, and (2) does it manifest in held-out scoring behavior. The first is a
data measurement question that can be answered by running a Wilcoxon test on the 69 pairs.
The second is a model behavior question answered by a Spearman correlation on held-out data.

The artifact anchor was corrected to include both `methodology_rationale.md` (where the
preference pair construction is described and the length check was never mentioned) and
`ablation_results.json` (Delta B = +0.3204, the result that cannot be interpreted without
ruling out the confound).

The adversarial pair test (Test 3) was added during sharpening as the minimal falsification
that does not require code — four hand-written emails that directly test whether rubric
beats length in the judge's scoring.
