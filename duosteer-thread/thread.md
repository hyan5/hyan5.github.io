# X thread — "Interpreting and Steering for Safe and Correct Code Generation"

Postable 6-post thread. Each post lists its figure under `figs/`. Post the figure
as a native image (not a link) and paste the alt text (see FIG_MAP.md) as the
image description.

---

## 1/6

📢 New Paper: "Interpreting and Steering for Safe and Correct Code Generation"

We make code LLMs write safer code without giving up correctness, at inference time, with no fine-tuning.

How: DuoSteer steers two directions at once, one for safety and one for code-correctness, at the attention heads that causally control each. Security improves without breaking working code.

🧵 What we found:

• Code safety is encoded in a small set of attention heads, not spread across the model
• Naive safety steering trades away correctness; DuoSteer steers both together and avoids it (-26.9% vulnerability, +7.5% correctness on average over 5 CWEs)
• It beats the practical baselines people actually use, prompting and supervised fine-tuning
• Holds on both Llama-3.1-8B and Qwen-2.5-Coder-7B

📄 https://arxiv.org/abs/2608.30025
🤗 https://huggingface.co/datasets/haaao821/CodeSec-Pairs

FIG: figs/post1_duosteer.gif

---

## 2/6

The data: CodeSec-Pairs, matched safe-vs-vulnerable Python code for the same task over 5 CWEs, labeled by CodeQL.

Two kinds of pairs:
• intra-prompt (same benign prompt, safe vs vulnerable by chance): trains the probes and steering vectors
• cross-prompt (vulnerability-eliciting vs benign prompt): for causal head localization

Llama-3.1-8B: 9,342 pairs (4,260 / 5,082). Qwen-2.5-Coder-7B: 2,500 (1,500 / 1,000). Plus safe-correct vs safe-incorrect pairs for the correctness direction.

FIG: figs/llama_data_stat.png

---

## 3/6

Where is "safe vs vulnerable" decided? Two lenses disagree.

Probing shows where it is encoded (linearly readable):

FIG: figs/fig_probe_combined_heatmap.png (probe accuracy per layer/head)

Causal knockout shows which heads actually cause it (blue = safe-promoting, red = vuln-promoting):

FIG: figs/fig_causal_heatmap.png (causal effect per layer/head)

The twist: they barely overlap. The causal heads are not the probe-salient ones (rank correlation near zero), so we steer at the causal heads.

---

## 4/6

Each direction is a mean-difference vector (safe minus vulnerable, and safe-correct minus safe-incorrect). At inference we add both to the causal heads, scaled by alpha. No retraining.

It helps most where a CWE's fix follows a consistent pattern, like unsafe deserialization and XSS, and less on harder ones like certificate validation. Overall it beats prompting, fine-tuning, and single-vector steering, and replicates on Qwen.

FIG: figs/results_slideshow.gif (cycles the per-CWE result tables: Llama then Qwen)

---

## 5/6

Why does the benefit vary by CWE? Two reasons.

Geometry: at the causal heads, the safety and correctness directions are near-orthogonal, which is what lets DuoSteer combine them.

Data: we label each pair by its fix pattern (substitute / guard / delete / unclear). The more concentrated a CWE's pattern, the better it steers. "Unclear"-dominated CWEs (path traversal, certificate validation) steer worst; this annotation signal rank-correlates with the reduction (Spearman rho ~= 0.90).

---

## 6/6

Huge thanks to my advisor @ZiyuYao for the guidance throughout this work.

📄 https://arxiv.org/abs/2608.30025
🤗 https://huggingface.co/datasets/haaao821/CodeSec-Pairs

Questions and feedback welcome!
