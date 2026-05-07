# Morning Call Summary — Day 3

**Partners:** Mistire Daniel (asker) + Day 3 partner (explainer)
**Topic:** Training and post-training mechanics

## What was ambiguous in the original draft

The original draft question asked broadly "why did my training fail?" The partner pushed
back: the question was mixing two distinct gaps — whether the *module selection* was wrong
and whether the *rank* was wrong. Answering both in one explainer would dilute both.

The second ambiguity: "output behavior unchanged" needed precision. The partner asked
whether Mistire meant the model produced identical token sequences or that rubric pass rates
were statistically identical. The answer (held-out pass rate 14.6% vs 14.6%, Delta A = 0.000,
p=0.585) made it the second — a measurable rubric outcome, not a vague sense of sameness.

## How the question was sharpened

The question was narrowed to module selection as the primary mechanism. Rank is a
secondary variable — acknowledged in the question but not the central ask. The partner
confirmed the final framing was unambiguous: the explainer would explain what q_proj and
v_proj specifically control in the attention mechanism, why updating only those matrices
might be insufficient to change constraint-following output behavior, and what module
combination would be appropriate for a rubric-scoring task.

The artifact anchor was made explicit: the tension between a decreasing training loss
(0.97→0.37 in `ablations/training_run.json`) and a flat held-out pass rate (Delta A = 0.000
in `model_card.md`) is the concrete observable the explainer must resolve — not the general
question of LoRA mechanics.
