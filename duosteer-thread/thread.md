# X thread — "Interpreting and Steering for Safe and Correct Code Generation"

Postable 6-post thread. Each post lists its figure under `figs/`. Post the figure
as a native image (not a link) and paste the alt text (see FIG_MAP.md) as the
image description. Every post is trimmed to fit the 280-character limit (URLs
count as 23 each; figures are media and do not count).

---

## 1/6

📢 New paper: Interpreting and Steering for Safe and Correct Code Generation

DuoSteer steers safety and correctness at once, at the heads that causally control each. No correctness loss, no fine-tuning.

-26.9% vulns, +7.5% correctness over 5 CWEs.

📄 https://arxiv.org/abs/2608.30025

FIG: figs/post1_duosteer.gif

---

## 2/6

The data: CodeSec-Pairs, matched safe vs vulnerable Python for the same task over 5 CWEs, labeled by CodeQL.

• intra-prompt: trains probes + steering vectors
• cross-prompt: for causal head localization

Llama: 9,342 pairs. Qwen: 2,500. Plus safe-correct vs safe-incorrect pairs.

FIG: figs/llama_data_stat.png

---

## 3/6

Where is "safe vs vulnerable" decided? Two lenses disagree.

Probing shows where it's linearly encoded. Causal knockout shows which heads actually cause it.

FIG: figs/fig_probe_combined_heatmap.png (probe accuracy) + figs/fig_causal_heatmap.png (causal effect)

The twist: they barely overlap (rank correlation near zero). So we steer at the causal heads.

---

## 4/6

Each direction is a mean-difference vector, added at the causal heads at inference, scaled by alpha. No retraining.

Works best where a CWE's fix is consistent (deserialization, XSS), less on cert validation. Beats prompting, SFT, and single-vector steering. Replicates on Qwen.

FIG: figs/results_slideshow.gif (per-CWE result tables: Llama then Qwen)

---

## 5/6

Why does the benefit vary by CWE? Two reasons.

Geometry: at the causal heads, safety and correctness are near-orthogonal, so DuoSteer can combine them.

Data: we label each pair's fix pattern. The more concentrated it is, the better it steers (Spearman rho ~0.90).

---

## 6/6

Huge thanks to my advisor @ZiyuYao for the guidance throughout this work.

📄 https://arxiv.org/abs/2608.30025
🤗 https://huggingface.co/datasets/haaao821/CodeSec-Pairs

Questions and feedback welcome!
