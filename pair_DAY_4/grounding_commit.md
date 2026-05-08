# Grounding Commit — Day 4

## Artifact edited

`week-11/tenacious-bench/model_card.md`

## What changed

**Limitations section, item 7 — quantified the power statement:**

Before:
> Held-out size: 41 tasks limits statistical power. Wide confidence intervals (±12pp) make
> small effects undetectable.

After:
> Held-out size: 41 tasks limits statistical power. At baseline pass rate 14.6%, this
> benchmark can only reliably detect improvements larger than 18pp at 80% power (α=0.05).
> A genuine 10pp adapter improvement would be detected in fewer than half of repeated
> experiments at this sample size. The correct interpretation of Delta A = 0.000 is not
> "no effect" but "no detectable effect at this N" — the two are not equivalent.

**Per-failure-family table — added exploratory note:**

Before:
> [table with no annotation]

After:
> [table] + footnote: "n=7–16 per family. These deltas are exploratory observations and
> cannot be distinguished from noise at standard significance thresholds (Fisher's exact
> p > 0.5 for F2 and F4). Treat as hypotheses for targeted retraining, not evidence of
> learned capability."

## Why this edit

Before Day 4, the model card made two claims that together implied a stronger conclusion
than the data supports: "Delta A is flat" (true) and "adapter didn't work" (one of several
interpretations consistent with a flat Delta A at n=41). The edit separates the observation
(flat Delta A) from the interpretation (underpowered evaluation) and prevents a reader from
treating the per-family directional deltas as findings.

The F2 and F4 annotations are the most important change — those rows will be the first
thing a reviewer cites as promising signal. Without the note, they look like real results.
With the note, they correctly redirect toward a targeted retraining experiment rather than
a deployment decision.
