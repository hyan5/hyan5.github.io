# X thread — "Interpreting and Steering for Safe and Correct Code Generation"

Postable 6-post thread. Each post lists its figure under `figs/`. Post the figure
as a native image (not a link) and paste the alt text (see FIG_MAP.md) as the
image description.

---

## 1/6

📢 New Paper: "Interpreting and Steering for Safe and Correct Code Generation"

We steer code LLMs to write safer code without giving up correctness, at inference time, with no fine-tuning.

Across 5 vulnerability types, our method DuoSteer cuts the vulnerability rate by 26.9% and improves correctness by 7.5% on average, beating prompting and fine-tuning.

Why it works: whether the model writes safe or vulnerable code is encoded in a small set of attention heads, so steering those heads is sufficient.

What we found 🧵

• Code safety lives in a small set of attention heads, not spread across the model
• Steering those heads removes vulnerabilities with zero retraining
• Safety usually costs correctness. DuoSteer steers both together and avoids that trade-off
• Holds on both Llama-3.1-8B and Qwen-2.5-Coder-7B

📄 https://arxiv.org/abs/2608.30025
🤗 https://huggingface.co/datasets/haaao821/CodeSec-Pairs

FIG: figs/post1_duosteer.gif

---

## 2/6

First, the data. CodeSec-Pairs is a set of matched safe-vs-vulnerable Python pairs across 5 CWEs (path traversal, XSS, code injection, improper certificate validation, unsafe deserialization), each pair solving the same task with safe/vulnerable labels from CodeQL.

Two ways we pair them:
• intra-prompt: both sides come from the same benign prompt (the model sometimes writes safe code, sometimes vulnerable). Used to train the probes and steering vectors.
• cross-prompt: the vulnerable side comes from a vulnerability-eliciting prompt and the safe side from the benign one, for the same task. Used for causal head localization.

Llama-3.1-8B: 9,342 pairs (4,260 intra + 5,082 cross). Qwen-2.5-Coder-7B: 2,500 pairs (1,500 intra + 1,000 cross).

For correctness we build a second kind of pair, safe-and-correct vs safe-but-incorrect for the same task, constructed at runtime from the model's own generations on top of the safety pairs.

FIG: figs/llama_data_stat.png

---

## 3/6

Where inside the model is "safe vs vulnerable" decided? Two lenses give different answers.

Probing: a linear classifier at every layer and attention head shows where the distinction is encoded, i.e. linearly readable.

FIG: figs/fig_probe_combined_heatmap.png (probe accuracy per layer/head)

Causal knockout: perturbing each head shows which heads actually cause the safe-vs-vulnerable outcome (blue = safe-promoting, red = vuln-promoting).

FIG: figs/fig_causal_heatmap.png (causal effect per layer/head)

The twist: these two views barely overlap. The heads that most cause safe-vs-vulnerable behavior are not the ones a probe would flag (rank correlation near zero). So we steer at the causal heads, not the probe-salient ones, and localize a code-correctness direction the same way.

---

## 4/6

DuoSteer applies two directions at those heads at inference, safety and code-correctness, at the same time.

Each direction is just the mean difference between the safe and vulnerable representations at a head (and safe-correct vs safe-incorrect for the correctness one). At inference we add it, scaled by a strength alpha, to that head's output. Nothing is retrained.

Where does it help most? On CWEs like code injection, unsafe deserialization, and cross-site scripting, DuoSteer gives the clearest vulnerability reductions with little correctness cost. The gains are more modest on others, such as improper certificate validation, where the detectable vulnerability rate is harder to shift.

Overall it moves the safety-correctness trade-off in the right direction, ahead of prompting, fine-tuning, and single-vector steering, and the pattern replicates on Qwen.

FIG: figs/results_slideshow.gif (cycles the per-CWE result tables: Llama then Qwen)

---

## 5/6

Post-hoc, why does DuoSteer's benefit vary by CWE? Two accounts.

Geometry: at the causal heads, the safety and correctness directions are near-orthogonal on every CWE, which is exactly what lets DuoSteer combine them, and the gain is largest where they are not aligned.

Data pattern: we label every pair by how the safe and vulnerable versions differ, its structural distance (a minimal edit / a refactor / a full rewrite) and its fix mechanism (substitute a dangerous API, add a guard, delete the unsafe call, or unclear). The more concentrated a CWE's fix pattern, the better it steers: substitution-dominated unsafe deserialization and guard-dominated XSS steer best, while "unclear"-dominated path traversal and certificate validation steer worst (this annotation signal rank-correlates with the reduction, Spearman rho ~= 0.90).

---

## 6/6

Huge thanks to my advisor @ZiyuYao for the guidance throughout this work. [add co-authors / funding / lab handle]

📄 https://arxiv.org/abs/2608.30025
🤗 https://huggingface.co/datasets/haaao821/CodeSec-Pairs

Questions and feedback welcome!
