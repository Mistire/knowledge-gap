# Sign-Off — Day 3

**Asker:** Mistire Daniel
**Gap-closure judgment:** Closed

## What I understand now that I did not before

Before this explainer I knew that my Week 11 training run "failed" — loss went down, held-out
stayed flat — and I attributed that to backbone size and a bad ablation design (evaluating a
judge as a generator). Both of those are true, but they are downstream of the actual
mechanism. The binding constraint was the module selection.

I now understand that `q_proj` and `v_proj` control attention routing and extraction — what
the model looks at and what it pulls out of context. They do not control the FFN layers where
constraint rules and factual associations are stored. A LoRA adapter that updates only q and
v can change style (which is why it works for generation tasks) but cannot inject or reinforce
rules the backbone hasn't encoded (which is why it failed on my constraint-following task).

The specific answer to my question "why did loss fall but held-out not move" is: the adapter
shifted attention patterns enough to prefer the chosen tokens in the 90 training pairs
(reducing SimPO loss), but could not change the FFN-encoded rule behavior that determines
rubric compliance on out-of-distribution adversarial probes. These decouple completely when
the adapter is updating the wrong component.

The concrete consequence: my model card Limitations section says "No multi-turn capability"
and "Rubric-dependent" but is silent on module selection. That is an underspecified
limitation. The correct next experiment is retrain with
`target_modules=["q_proj","k_proj","v_proj","o_proj","gate_proj","up_proj"]` at r=8
and re-run the held-out ablation.

## Grounding commit

See grounding_commit.md — one paragraph added to `methodology_rationale.md` and one row
updated in `model_card.md`.
