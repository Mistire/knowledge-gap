# Evening Call Summary — Day 3

**Partners:** Mistire Daniel (asker) + Gersum Asfaw (explainer)
**Direction:** Gersum's explainer → Mistire's question (q_proj/v_proj module selection)

## Feedback Mistire gave on the explainer

The explainer landed on the core mechanism: that constraint-following behavior lives in
the FFN layers and updating only q_proj and v_proj cannot reach it. That closed the gap.

Two pieces of feedback triggered revisions:

1. The explanation of *why the training loss fell* was initially too brief — one sentence
   linking loss to within-distribution preference separation. Mistire pushed back: "I need
   to understand why loss and held-out can decouple, not just that they can." Gersum added
   the paragraph distinguishing what SimPO's loss measures (log-prob margin on training
   pairs) from what the held-out measures (OOD generalization on adversarial probes).

2. The "what would have worked better" section originally listed modules without explaining
   the role of each. Mistire asked: "why gate_proj and up_proj specifically — why not
   down_proj?" Gersum added the Geva et al. pointer and revised the table to include the
   role column.

## What the writer revised

Both paragraphs above were expanded in the post-call revision. The module table was
restructured with a "Why include for this task" column. The Geva et al. (2021) reference
was added to the pointers section.

## Gap closure assessment

See signoff.md.
