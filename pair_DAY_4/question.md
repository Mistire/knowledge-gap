# Day 4 Question — Evaluation and Statistics

**Asker:** Mistire Daniel
**Topic:** Evaluation and statistics — statistical power and null results in small-N benchmarks
**Date:** Day 4, Week 12

## The Question

My Week 11 model card (`model_card.md`, evaluation table) reports Delta A = +0.000 with a
95% CI of [−0.122, +0.122] and p = 0.585. I wrote this up as "not significant" and listed
"41 tasks limits statistical power" as limitation #7. But I also reported per-failure-family
deltas in the same table: F2 improved by +12.5% and F4 improved by +14.3%.

What I didn't notice until writing this question: the F2 row has n=8 tasks and the F4 row
has n=7. A 12.5% improvement on n=8 is exactly one task changing from fail to pass. A 14.3%
improvement on n=7 is the same thing.

**What does a p-value of 0.585 and a ±12pp confidence interval actually tell me about whether
my adapter changed anything — and given that my per-family subsets have n=4 to n=16, how many
tasks would I have needed in each family to confidently detect a 10–15% improvement in pass
rate if one existed?**

Specifically: what statistical test produces a p-value for a pass-rate comparison (is it a
proportion z-test, Fisher's exact, something else)? What is the minimum detectable effect
size (MDE) for a two-proportion test at n=41 with 80% power? And for the per-family results —
what is the probability that a genuine +12.5% improvement on F2 would fail to reach
significance at n=8? Does "not significant" mean "no effect" or "underpowered to see the
effect at this sample size"?

## Connection to Existing Work

Knowing this would let me revise:

- **`week-11/tenacious-bench/model_card.md` (evaluation table, rows Delta A and per-family
  breakdown)** — I concluded "Delta A is flat" without computing the minimum detectable effect
  at n=41. If the MDE at 80% power is ~28pp (plausible for n=41), then a genuine 10% adapter
  improvement would be completely invisible in this evaluation. The conclusion "adapter didn't
  work" and the conclusion "benchmark couldn't detect it" are both consistent with my data —
  I cannot currently distinguish between them.

- **`week-11/tenacious-bench/model_card.md` (Limitations, item 7)** — I wrote "Wide confidence
  intervals (±12pp) make small effects undetectable" without knowing what effect sizes are
  actually undetectable at this N. The limitation is correct but unquantified. A revised
  version would state the MDE explicitly: "At n=41, this benchmark can only reliably detect
  improvements larger than Xpp at 80% power."

- **`week-11/tenacious-bench/model_card.md` (per-family table, F2 and F4 rows)** — The +12.5%
  on F2 and +14.3% on F4 are presented without confidence intervals or per-row significance
  tests. At n=8 and n=7 respectively, these deltas cannot be distinguished from noise at
  any standard significance threshold. Either those rows need CIs, or they need a note
  explicitly stating they are exploratory observations, not findings.

## Why This Matters Beyond My Work

Any FDE building an evaluation benchmark for a fine-tuned model faces this problem: the
number of held-out tasks is usually small (tens, not hundreds) because labeling is expensive
and the task domain is narrow. The natural temptation is to run the evaluation, observe a
per-category breakdown, find a few positive deltas, and report them as signal. But without
knowing the minimum detectable effect at the subset's N, those per-category deltas are equally
consistent with random noise. Understanding the relationship between N, effect size, and
statistical power determines whether an evaluation can support the conclusions drawn from
it — and whether "null result" means "doesn't work" or "benchmark too small to tell."

## Four-Property Check

| Property | Status |
|---|---|
| Diagnostic | Names the specific gap: what statistical test applies to pass-rate comparisons, what the MDE is at n=41, and why per-family results at n=4–16 cannot be reported as findings without CIs |
| Grounded | Tied to `model_card.md` evaluation table (Delta A = 0.000, p=0.585, CI [−0.122, +0.122]), per-family rows F2 (n=8, +12.5%) and F4 (n=7, +14.3%), and Limitation #7 (power statement without quantification) |
| Generalizable | Every FDE running a small-N eval faces this — the "I saw a positive delta in one category" pattern is nearly universal and almost always underpowered |
| Resolvable | A colleague can answer this in 800 words with a power calculation for two proportions, the formula for MDE at 80% power, and a direct answer on whether my F2/F4 deltas are distinguishable from noise at their respective Ns |
