# Grounding Commit — Day 3

## Artifact edited

`week-11/tenacious-bench/methodology_rationale.md` and
`week-11/tenacious-bench/model_card.md`

## What changed

**`methodology_rationale.md` — Training Configuration table, Target modules row:**

Before:
> `q_proj, v_proj` | Attention layers only; smaller adapter footprint

After:
> Unsloth default for Qwen; optimized for generation-quality tasks per LoRA paper Table 6.
> **Underdefended for this task:** constraint-following behavior is stored in FFN layers
> (gate_proj, up_proj) per Geva et al. 2021. Updating only q and v changes attention routing
> but cannot inject domain rules the backbone has not encoded. A more appropriate set for a
> rubric-scoring adapter would include o_proj, gate_proj, and up_proj — this is a v0.2
> retraining target.

**`model_card.md` — Limitations section:**

A new Limitation 1 was inserted ("Module selection underdefended") before the existing
scale-asymmetry limitation. The existing limitations were renumbered 2–7 accordingly.
The new limitation names the mechanism (FFN rule storage per Geva et al. 2021), connects
it to the observed flat Delta A, and specifies the v0.2 retraining target module set.

## Why this edit

Before Day 3, both documents attributed the flat Delta A to "scale asymmetry" and "role
confusion." Both are real factors, but neither identifies the binding mechanism. The module
selection (q_proj + v_proj only) was the primary constraint: these matrices control attention
routing and extraction, not the FFN layers where constraint rules are stored. The adapter
could not generalize the Tenacious constraint families to held-out adversarial prompts
because the FFN layers encoding those rules were frozen throughout training.

The edit converts a vague "this is a limitation" acknowledgment into a mechanism-grounded
statement with a specific testable fix. A reviewer reading the model card now knows *why*
the adapter underperformed and what experiment would test whether the diagnosis is correct.
