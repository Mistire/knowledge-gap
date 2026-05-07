# Tweet Thread — Day 3

*Compressing Mistire's blog post: "LoRA Rank is a Production Decision, Not a Hyperparameter"*

---

**Tweet 1**

Your LoRA model card says rank=16. If someone asks why not 8 or 32, can you answer
with evidence — or just "I tried it and the loss looked okay"?

Rank is not a hyperparameter you tune until training converges. It is a capacity decision
you justify with a specific experiment. Here is how. 🧵

---

**Tweet 2**

Rank r controls how many independent directions in weight space the adapter can update.

ΔW = BA = b₁a₁ᵀ + b₂a₂ᵀ + ... + bᵣaᵣᵀ

r=1 → one axis of movement. r=64 → 64 independent axes.

More axes = more expressivity. But with 100 training pairs, more axes = more ways to
memorize rather than generalize.

---

**Tweet 3**

The tradeoff:

↓ rank too low: adapter cannot span the weight-space subspace the task needs.
Loss plateaus. Held-out flat.

↑ rank too high: adapter memorizes training pairs instead of generalizing.
Loss falls fast. Held-out still flat — for a different reason.

Both failure modes look like "training didn't work." They need different fixes.

---

**Tweet 4**

The minimal experiment that makes a rank defensible:

Train 5 variants: r = 1, 4, 8, 16, 32 — identical config, same seed.
Evaluate each on your dev partition (not held-out).
Plot: dev pass rate vs rank.

The curve will plateau. Pick the smallest r at the plateau.

That's the Pareto point: highest quality, lowest parameter count, lowest inference cost.

---

**Tweet 5**

Once you have the plot, your model card can say:

"r=16 sits at the quality plateau (r=32 adds +0.3pp within measurement noise) and adds
Y ms latency vs no adapter. r=8 misses F2 classification by 8pp — below the acceptable
threshold for the Tenacious deployment."

That is a production decision. "r=16 was in the starter notebook" is not.

---

**Tweet 6**

Full write-up: why rank mathematically constrains capacity, the under/over-capacity
failure modes, and the exact experiment to run on your existing tenacious-bench stack.

[blog post link]

Sources: LoRA paper Table 6 (Hu et al., ICLR 2022) + Intrinsic Dimensionality
(Aghajanyan et al., ACL 2021)
