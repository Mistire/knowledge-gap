# Sign-Off — Day 4

**Asker:** Mistire Daniel
**Gap-closure judgment:** Closed

## What I understand now that I did not before

Before this explainer I reported Delta A = 0.000 and p = 0.585 and concluded "the adapter
didn't improve the baseline." I also listed "41 tasks limits statistical power" as
Limitation 7 in the model card — but I wrote that without knowing what it actually meant
numerically or whether my conclusion was defensible.

I now understand three things I could not have said precisely before:

**1. "Not significant" does not mean "no effect."**
p = 0.585 means the data is consistent with no effect, but also consistent with a real
improvement of up to 12pp in either direction. The test has no power to distinguish these.
The correct interpretation is: "The evaluation cannot rule out effects anywhere in the
range [−12pp, +12pp]." My conclusion "adapter didn't work" is one of multiple explanations
consistent with the data.

**2. The minimum detectable effect at n=41 is ~18pp.**
At baseline 14.6% and n=41, my benchmark can only reliably detect improvements larger than
18 percentage points at 80% power. A genuine 10pp improvement would be detected in fewer
than half of repeated experiments. My evaluation was 34% of the sample size needed to
detect the effect I cared about. The evaluation was underpowered before I ran it.

**3. The per-family deltas are not findings.**
F2 (+12.5%, n=8) and F4 (+14.3%, n=7) are each one task changing from fail to pass.
Fisher's exact test on both returns p > 0.5. The MDE for F2 at n=8 is approximately +42pp.
Presenting these as directional signal without confidence intervals or significance tests
was misleading. The correct language is "exploratory observations, not findings."

## Grounding commit

See grounding_commit.md — Limitation 7 quantified and per-family table annotated in
`model_card.md`.
