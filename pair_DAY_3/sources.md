# Sources — Day 3

## Canonical sources used in the explainer

1. **Hu, E. J., Shen, Y., Wallis, P., Allen-Zhu, Z., Li, Y., Wang, S., ... & Chen, W. (2021).
   LoRA: Low-Rank Adaptation of Large Language Models.**
   *arXiv:2106.09685 / ICLR 2022.*
   https://arxiv.org/abs/2106.09685
   — Primary source for the ΔW = BA decomposition, the intrinsic dimensionality motivation,
   and the original ablation over which weight matrices to target (Table 6: Wq+Wv outperforms
   all other pairs on generation tasks, but the paper does not test constraint-following tasks).

2. **Aghajanyan, A., Zettlemoyer, L., & Gupta, S. (2020).
   Intrinsic Dimensionality Explains the Effectiveness of Language Model Fine-Tuning.**
   *arXiv:2012.13255 / ACL 2021.*
   https://arxiv.org/abs/2012.13255
   — The theoretical foundation for why low rank works: fine-tuning objectives have low
   intrinsic dimensionality, meaning the gradient update subspace is much smaller than the
   full parameter space. This is the "why" behind LoRA's core premise.

## Tool / pattern used

**Experiment:** Inspected Unsloth's default LoRA target module selection for Qwen models
(`target_modules = ["q_proj", "v_proj"]`) against the LoRA paper's Table 6 ablations, and
cross-referenced with the Qwen 2.5 architecture (32 attention heads, hidden dim 896 for 0.5B)
to compute the adapter parameter count at r=16:
- q_proj: 896×896 → LoRA adds 2 × (896×16) = 28,672 parameters
- v_proj: 896×896 → same
- Total LoRA parameters: ~57K per layer × 24 layers ≈ 1.4M trainable parameters
- Full model parameters: ~494M
- LoRA fraction: ~0.28% of parameters updated

This concrete count makes visible why rank and module selection interact: at 0.28% of
parameters, the adapter has very limited expressive capacity — sufficient for style transfer
(generation tasks) but potentially insufficient to override knowledge-retrieval behavior
encoded in the FFN layers.
