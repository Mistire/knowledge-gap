# LoRA Rank is a Production Decision, Not a Hyperparameter

*Written by Mistire Daniel for Gersum Asfaw's Day 3 question*

---

Gersum's Week 11 model card reports a LoRA rank `r`, lists quality outcomes on
tenacious-bench, and names the rank in the training configuration table. It cannot defend
that number. If a client engineer asks "why `r=X` and not `r=X/2`?" the honest answer is:
we tried it and it worked. That is not a production decision. That is a benchmark outcome
without a mechanism.

This post closes the gap: what rank mathematically constrains, why that creates a
specific under/over-capacity tradeoff, and what the minimal experiment is to justify a rank
choice as defensible for the Tenacious deployment.

## What Rank Actually Controls

LoRA adapts a weight matrix W₀ by adding ΔW = BA, where B ∈ ℝ^{d×r} and A ∈ ℝ^{r×k}.
The rank r is the number of *independent directions* in weight space the adapter can update.

A concrete way to see this: ΔW = BA is a sum of r rank-1 outer products —

```text
ΔW = b₁a₁ᵀ + b₂a₂ᵀ + ... + bᵣaᵣᵀ
```

Each term shifts the weight matrix along one direction. r=1 means you can only move the
matrix along a single axis — very constrained. r=64 means you can combine 64 independent
directions — much more expressive, but also much more prone to fitting noise when your
dataset is small.

This is not an abstract concern. The number of trainable parameters scales linearly with r:
for a Qwen 0.5B layer (hidden dim 896), one LoRA module at r=16 adds 2 × (896 × 16) = 28,672
parameters. Doubling to r=32 exactly doubles that. Across 24 layers and all targeted modules,
the difference in trainable parameter count between r=8 and r=64 is roughly 8×.

## The Tradeoff: Under-Capacity vs Over-Capacity

**Under-capacity (rank too low):** The adapter cannot span the subspace of weight-space
changes needed to learn the target behavior. Training loss may plateau early or fail to
decrease at all. Held-out performance will be worse than the baseline because the adapter
has learned a distorted but still-underfit approximation.

**Over-capacity (rank too high):** The adapter has enough expressivity to memorize the
training examples rather than generalizing the underlying pattern. Training loss falls fast
and deep. Held-out performance degrades or stays flat because what was learned is the
specific token sequences in training pairs, not the generalizable behavior. With 102 pairs,
this risk is real above r=32 for most module configurations.

The key insight from Aghajanyan et al. (2021): fine-tuning tasks have *low intrinsic
dimensionality*. The gradient updates that matter during fine-tuning naturally concentrate
in a subspace far smaller than the full weight space. For generation-style tasks, r=4 often
matches r=64. For constraint-following tasks with five interacting constraint families, the
intrinsic dimension is higher — but still bounded. The question is not "bigger rank is
better" but "what is the intrinsic dimension of this specific task?"

## The Minimal Experiment to Justify a Production Rank

You do not need a full hyperparameter sweep. You need a *rank ablation* on your existing
stack evaluated on the tenacious-bench dev partition (not the sealed held-out):

```python
ranks_to_test = [1, 4, 8, 16, 32]
```

For each rank: train with identical configuration (same epochs, LR, seed, module set,
training data). Evaluate on the dev partition using your existing `scoring_evaluator.py`.
Record three numbers per rank:

| Metric | Why it matters |
| --- | --- |
| Dev pass rate (%) | Quality signal — is more rank actually helping? |
| Adapter parameter count (M) | Size cost — proportional to r |
| Inference latency delta (ms p95) | Runtime cost — measured on your existing eval loop |

Plot pass rate vs rank. The curve will plateau — it always does. The right production rank
is the smallest r at which the curve has flattened. Everything to the right is paying a
latency and size cost for no quality gain.

This experiment costs nothing beyond the GPU time you already have. On a Colab T4 with
Unsloth, five training runs at the ranks above take roughly 10–40 minutes total. The
resulting plot is the artifact that converts "I used r=16" into "I used r=16 because it
sits at the quality plateau and r=32 adds 15% adapter overhead for +0.3pp pass rate that
is within measurement noise."

## What "Defensible for Production" Looks Like

A rank is defensible for production when you can make three statements with evidence:

1. **Quality claim:** "Ranks above r=X show no statistically meaningful improvement on
   the tenacious-bench dev set (pass rate within ±2pp at the same seed)."

2. **Cost claim:** "Each doubling of rank adds approximately Y ms to p95 inference latency
   and Z MB to the adapter checkpoint, based on measured runs on [hardware]."

3. **Pareto claim:** "r=X sits at the quality plateau with the lowest cost footprint —
   any lower rank misses [specific failure family], any higher rank adds cost with no
   quality gain."

Without these three statements, a rank in a model card is a benchmark number. With them,
it is a production decision a client engineer can interrogate and a future FDE can revisit
when the training data changes.

## Applied to the Tenacious Conversion Engine

Gersum's conversion engine has a LoRA component contributing to quality and cost outcomes
across the `enrich → compose → qualify → book → sync` pipeline. The rank ablation above
produces:

- A revised model card section that names the quality plateau and cites the experiment
- A deployment recommendation that can quote per-call latency delta at the chosen rank
- A v0.2 retraining trigger: if the training data grows past ~500 pairs, re-run the
  ablation, because the intrinsic dimension of the task may shift and the current rank
  may become underpowered

The rank you trained with may turn out to be the right answer. The experiment makes that
finding defensible rather than accidental.

## Pointers

- **LoRA paper** (Hu et al., ICLR 2022), especially Table 6 and Appendix F.2 on rank
  sensitivity — shows that for many tasks r=4 matches r=64.
- **Intrinsic Dimensionality** (Aghajanyan et al., ACL 2021) — the empirical foundation
  for why rank can be small.
- **Unsloth Qwen 3.5 fine-tuning guide** — documents the default module set and explains
  the memory/speed tradeoff at different ranks on T4 hardware.
- Experiment to run: the five-rank ablation above on your existing tenacious-bench stack.
  One afternoon. One Pareto plot. One defensible model card.
