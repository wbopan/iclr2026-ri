We thank the reviewer for the thoughtful and constructive feedback. We are encouraged that the reviewers find RI captures a fundamental aspect of calibration and appreciate our strong experiments. We carefully address your concerns point by point below.

> **W1**: The mathematical presentation of RI feels dense. Figure 1 hints that the Refusal Index captures convexity of the curve, and further developing this intuition and/or motivating the Gaussian copula fit could aid clarity

Thank you for raising this, and we agree that Figure 1 can be made more accessible to strengthen our claim. In the original manuscript, we have provided the intuition behind RI and the accuracy–refusal trade-off in a paragraph around L320, with more technical derivation in Appendix E. **To make the math more intuitive, in the revision, we now include references to the Discussion section in Figure 1's caption.**

> **W2 & Q3**: Is RI measuring something fundamentally different from sampling-based calibration methods, or is it primarily a more sample-efficient approximation?

Thank you for this important question.
Conceptually, RI and sampling-based rank-calibration target the same underlying quantity: how well the model's refusal behavior aligns with incorrectness. The key differences are:

**How refusal probability is estimated.** Sampling-based methods explicitly estimate prediction probabilities from multiple samples under temperature=1 and estimate refusal probability using refusal frequency. However, it's unknown how accurate this estimate is. In comparison, RI directly estimates the rank correlation from observable binary outcomes of refusal and error.

**Efficiency.** In Section 3.3, we compare RI with AUROC computed from 100 samples per question using P(Answering) as in SimpleQA. RI achieves high correlation with this AUROC while requiring only 2 generations per question, compared to 100.

We agree that a clearer presentation would be beneficial. **In the revision, we have followed your suggestion by adding a table that directly compares RI with other calibration methods to better highlight RI's advantages.** Please see the table below:

| Method                   | Typical calibration method(s)                       | Unbiased to true refusal probability? | Computational cost                                                    |
| ------------------------ | --------------------------------------------------- | ------------------------------------- | --------------------------------------------------------------------- |
| Linear Probe             | Train a linear classifier on internal hidden states | ✗                                     | $S*N_\text{train}*d$ Probe training + $N$ generations + $N$ inferences                 |
| Black-box Estimator      | Train an auxiliary classifier on output text        | ✗                                     | Calibrator training on $N_\text{train}$ samples + $N$ generations + $N$ inferences |
| Verbalized Confidence    | Ask model to output a numeric confidence value      | ✗                                     | $N$ generations + $N$ confidence-score generations                        |
| Sampling-based           | Use refusal frequency to approximate refusal prob.  | ✓                                     | $S*N$ generations (S: samples per question)                         |
| **Refusal Index (ours)** | Two-pass evaluation, no extra model                 | ✓                                     | $2N$ generations                                                    |

> **W3**: Gemma-3-12b's Refusal Index varies considerably across prompts (roughly 0.1–0.3). Is Gemma an outlier? How stable is RI in practice?

Thank you for pointing this out. In the original manuscript, we include all raw results from SimpleQA in Appendix F. In our experiments, Gemma3-12B behaved as an outlier, with less consistent adherence to the system instruction than the other models. Therefore, we did not include it in the stability test initially. We hypothesize that this may be related to its smaller model size and corresponding limitations in following system prompts consistently, which we have listed as one of the failure modes of RI in Appendix A.

We agree that showing results for more models would strengthen our claims. **In the revision, we have updated the table to include results for all models tested.** While Gemma3 shows a higher coefficient of variation on RI than other models, RI still exhibits the lowest variance compared to other metrics.

> **W4 & Q2**: It's not possible to identify which specific model corresponds to each data point in the frontier model evaluation figure.

Thank you for this valuable suggestion. In the original manuscript, we have included all raw results from SimpleQA in Appendix F. To make the figures easier to interpret, **in the revision, we have followed your suggestion and updated Figure 5 by adding name labels to each data point.**

> **Q1.1**: How is stability computed in Table 2? Could differences in distribution concentration affect the apparent variability of different metrics?

Thank you for asking us to clarify this. In Table 2, we report two complementary stability measures across the four refusal prompts for each model:

- The **normalized difference**

    $$
    \Delta \text{Metric} = \frac{\text{Metric}_\text{max} - \text{Metric}_\text{min}}{\lvert \text{Metric}_\text{mean} \rvert}
    $$
    
    which captures how far the most and least stable runs deviate relative to the average level of the metric.
    
- The **coefficient of variation (CV)**
    
    $$
    \text{CV} = \frac{\text{standard deviation}}{\lvert \text{mean} \rvert}
    $$

    which measures relative dispersion around the mean and is standard for comparing variability across metrics with different scales.

Both metrics are explicitly scale-normalized, which controls for differences in how concentrated the underlying distributions are. **In the revision, we have updated Section 3.2 to include these formulas.**

> **Q1.2**: What level of RI variation across prompts should be considered acceptable?

While our main claim is that RI shows much less variance than other metrics, we don't rely on cross-prompt variance when using RI for evaluation: when comparing RI across different models, we evaluate those models under the same system prompt, so variance across prompts doesn't affect the comparison. For variance within a single prompt, in Appendix C, we provide ablation experiments on how many samples are needed for acceptable variance: around 1200 samples.

> **Q1.3**: Would it be possible to show empirical data points on iso-RI curves (as in Figure 3) for more models?

We agree that this is a very helpful visualization. **In the revision, we have extended the iso-RI visualization to include more models, including Gemma-3-12B, Qwen3-32B, Qwen3-32B-Think, Qwen2.5-72B, DeepSeek-V3, and Mistral-123B**. For each model, we plot the empirical (refusal rate, correct answer rate) pairs induced by the four prompts. This addition makes it clear which models have tightly clustered points that align with a single iso-RI curve and which models show more variation. Please see the figure below.

![iso-ri](https://cdn-uploads.huggingface.co/production/uploads/62cd3a3691d27e60db0698b0/466Xck-WTlF9uGrco01RP.png)

We hope these clarifications and the additional experimental evidence address your concerns. Please let us know if you have any further questions!
