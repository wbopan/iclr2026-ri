We thank the reviewer for their thoughtful feedback and constructive suggestions. We carefully address each concern below.

> **W1:** The paper lacks sufficient explanation of traditional evaluation metrics.

Thank you for raising this point. We agree that this background is essential for understanding our contribution. In the original submission, traditional metrics were mentioned in the Introduction and listed in Table 1. In the revision, **we have added a dedicated "Background" section after the Introduction**, where we (1) introduce traditional metrics in more detail, relocating Table 1 to this section; and (2) explain why these metrics are insufficient for measuring knowledge‑aware refusal. In summary:

- **Refusal Rate:** This metric captures only the **frequency** of refusals, not the **quality** of those decisions(i.e. whether refusals correlate with errors). Moreover, a model's refusal rate can be trivially manipulated through prompts encouraging cautious behavior.
- **Heuristic Combinations (e.g., F-score):** These metrics combine refusal and correctness in ways primarily designed to penalize over-refusal. We show that such combinations, including the SimpleQA F‑score [3], can vary by up to ~70% when only the refusal tendency is changed, demonstrating that they conflate accuracy with refusal rate rather than measuring the alignment between them.

> **W2 & W5:** Why not estimate confidence and threshold it (e.g., via AUROC) instead of measuring refusal explicitly?

Thank you for this insightful question, which touches on a core aspect of our motivation. We understand your concern to be: why not use confidence scores and threshold them to approximate refusal behavior?

The fundamental distinction is that **calibration** and **knowledge‑aware refusal** are related but different:
- Calibration methods produce numeric probabilities of correctness and evaluate how well those probabilities match actual accuracy.
- Knowledge‑aware refusal (RI) evaluates how well the model's **actual refusal decisions** align with the probability of being incorrect.

**The challenge with external calibrators:** External calibrators cannot replace direct refusal measurements because they often **do not reflect the actual refusal decisions made by models**. Even if we instruct or train models to output confidence scores, thresholding those scores does not necessarily approximate the model's real refusal behavior.

We support this claim in two ways in the revised paper: (1) evidence from prior research, and (2) a new ablation experiment.

- **Prior work shows strong disagreement among calibrators**: SimpleQA [3] finds that verbalized confidence is severely over‑confident, while sampling‑based $P(\text{Answering})$ is moderately well‑calibrated; while APRICOT [2] can appear nearly perfectly calibrated. [5] points out that it becomes unclear which results to trust, as they rely on different assumptions of calibrators and differ greatly in scores.
- **New ablation experiment**: To empirically validate this point, we added an ablation study using Qwen3‑32B, comparing three estimators:
	- $P(IK)$: White-box linear probe confidence estimator [1]
	- `APRICOT`: Accurate black-box confidence estimator [2]
	- $P(\text{Answering})$: Sampling-based refusal frequency method [3], which makes the fewest assumptions about calibration and has been shown to align with RI

	**![View comparison of calibration methods](https://cdn-uploads.huggingface.co/production/uploads/62cd3a3691d27e60db0698b0/vzzdjNXM2nF3Kxh3gnldi.png)**

	The results show that their calibration curves and ECE values differ substantially. Only $P(\text{Answering})$ aligns well with RI, yet RI reaches similar conclusions using just two evaluation passes instead of 100 samples per question. We have incorporated these findings into the Background section of the revision.

> **W3:** Clarification on error definitions and Table 1 formulas. Are refusals also counted as errors? The formula c/(1–r) is unclear — does c represent the number of correct answers among the non-refused cases?

Thank you for identifying this ambiguity. We clarify as follows:

- **Refusals as errors:** Yes, refusals are counted as "incorrect" in our formulation. As stated in Section 3.1: "we define two indicators: $W_i = \mathbf{1}\{f_{\text{LM}}(x_i) \neq y_i\}$ for incorrect outputs and $R_i = \mathbf{1}\{f_{\text{LM}}(x_i) = \bot\}$ for refusal responses." Since $\bot \neq y_i$, all refusals are counted as incorrect. We have made this more explicit in the revised text.
- **Re-answering:** The re-answering process, which checks whether a refusal was warranted, is captured by $W'_i$, as described in the methodology.
- **Table 1 notation:** Regarding the formula $c/(1-r)$, note that **$c$ represents the global correct answer rate** (the proportion of correct answers among _all_ samples), as indicated in the first row. This differs from $C/A$, which measures accuracy only among answered questions. **We have revised the Table 1 caption to clarify this distinction**.

> **W4**: RI seems closely related to AUROC. How does it differ, and why use RI instead?

We agree that RI shares conceptual similarities with AUROC—both are rank‑based discrimination metrics. RI should be understood as a **refusal‑specific, sample‑efficient analogue** of AUROC.

The key differences are:
- AUROC requires a **real‑valued confidence score**, which for black‑box LLMs must be constructed using a chosen calibrator (e.g., verbalized confidence, linear probe, sampling, APRICOT). The resulting AUROC therefore evaluates the quality of that particular score.
- RI is defined as the **Spearman correlation between latent refusal probability and error probability**, estimated **directly from binary refusal decisions and correctness labels** via a Gaussian copula, without requiring any auxiliary calibrator.

Empirically, RI is highly correlated (≈85%) with AUROC computed from the sampling‑based $P(\text{Answering})$ method [3], yet requires only two evaluation passes rather than 100 samples per question (Figure 4). Thus, RI captures essentially the same discriminative structure as a strong sampling‑based calibration approach while being far more practical to deploy [3,5].

> **W6:** Robustness of the Gaussian estimation under limited sample sizes.

Thank you for raising this methodological concern.

- **Gaussian copula vs. Gaussian distribution:** We use a Gaussian _copula_ to model the dependency structure (correlation) between refusal and error. Critically, this does _not_ assume that the marginal distributions of refusal or error are Gaussian—it only models their correlation structure. This approach is standard practice for estimating tetrachoric correlation [4].
- **Goodness-of-fit:** To empirically validate this choice, we provide an **ablation experiment in Appendix G** comparing the Gaussian copula against other copula families (e.g., Clayton, Student-t). The results demonstrate that the Gaussian copula consistently provides stable and superior fit for this task. We have clarified this distinction and added the relevant citation in the revision.

> **Q7:** The choice of baselines could be further discussed.

Thank you for this question.

- **Binary confidence metrics:** The metrics in Table 1 (Accuracy, C/A, F-score) represent the standard baselines currently used in factual QA evaluation, particularly in SimpleQA [3].
- **Fine-grained confidence metrics:** We compare against AUROC computed with $P(\text{Answering})$ because it is the most robust rank-based metric available for this task [3]. We excluded ECE because it measures absolute calibration error rather than the discriminative ranking ability that RI is designed to assess.

**In the revision, we have relocated Table 1 to the Background section and expanded the discussion to explicitly justify these baseline choices.**

> **Q8:** Impact of accuracy/model competence and authors' perspective on how model competence affects alignment.

This is an excellent question. One might expect that highly capable models would trivially "solve" refusal, but our findings suggest otherwise.

- **Independence from accuracy:** As shown in **Table 3**, RI maintains high ranking stability even after we mathematically remove the effects of accuracy using isotonic regression. In contrast, heuristic metrics like F-score degrade to near-random performance when accuracy effects are controlled for. This demonstrates that RI measures the _quality of the refusal mechanism itself_, not merely the model's overall competence.
- **Family vs. capability:** **Figure 5** shows that refusal alignment (RI) does not scale linearly with accuracy. Some highly accurate model families still exhibit poor refusal alignment. This supports a key insight of our work: knowledge-aware refusal appears to result from specific training methodologies rather than being an automatic byproduct of general capability.

We hope these clarifications and the additional experimental evidence fully address your concerns. Please let us know if you have any further questions!

**References:**

[1] Kadavath, Saurav, et al. "Language models (mostly) know what they know." _arXiv preprint arXiv:2207.05221_ (2022).

[2] Ulmer, Dennis, et al. "Calibrating Large Language Models Using Their Generations Only." _ACL_ (2024).

[3] Wei, Jason, et al. "Measuring short-form factuality in large language models." _arXiv preprint arXiv:2411.04368_ (2024).

[4] Olsson, U. "Maximum likelihood estimation of the polychoric correlation coefficient." _Psychometrika_ 44, 443–460 (1979).

[5] Huang, Xinmeng, et al. "Uncertainty in Language Models: Assessment through Rank-Calibration" EMNLP (2024)
