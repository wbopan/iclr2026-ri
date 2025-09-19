**Page 1:**
Factuality Calibration Estimation from Refusal
Methodology and Experiments
Wenbo Pan
June 16, 2025

**Page 2:**
Table of Contents
Introduction
Methodology
Experiment

**Page 3:**
Changelogs - Since 24th May
• Large-scale evaluation on SimpleQA and TriviaQA.
• Multiple baseline calibration methods (accurate but expensive) to compare with CER.
• Downplay the concept of hallucination in the paper; instead, use "factuality".

**Page 4:**
Introduction
Challenges of Factuality Calibration
• An important metric to measure factuality in LLMs is calibration.
• Calibration measures the extent to which a large language model's (LLM's) predicted probabilities match actual accuracy.
• It is difficult to measure the calibration level of LLMs directly from text output.
• Previous works train auxiliary models to estimate the calibration level of LLMs.

**Page 5:**
Introduction
Our Paper
• We propose a new method called CER to estimate the inherent calibration of LLMs simply from the accuracy and refusal rate on a set of questions.
• Compared to other metrics that use refusal rate to estimate factuality, CER is more interpretable and stable.
• Despite being computationally lightweight, CER aligns well with more expensive and accurate calibration estimation methods.
• Finally, we use CER to study the factors impacting factuality in LLMs.

**Page 6:**
Methodology
Definition of CER
• Consider a factual question answering setting.
• An answer provided by the LLM is either correct or incorrect.
• Alternatively, the LLM can refuse to answer.
• A more factually calibrated LLM can accurately refuse to answer when a question is beyond its knowledge, but answer confidently when it is likely to be correct.
• As a result, refusal rate is considered together with accuracy to estimate the factuality calibration of LLMs.

**Page 7:**
Methodology
Definition of CER
• We define CER as the correlation between "confidence" and "accuracy" of the LLM.
• Confidence is defined as the "perceived accuracy" of the LLM given the actual accuracy of an answer, i.e., confidence = f(accuracy).
• We assume the model will refuse to answer when confidence < τ.
• CER is defined as the correlation between confidence and accuracy.

**Page 8:**
Methodology
The Latent Confidence Hypothesis
• To support this definition, we propose a latent confidence hypothesis:
• The probability of the LLM refusing to answer a question is P(refuse) = P(c < τ), where c ~ FC(a, ρ).
• FC is the distribution function of the confidence parameterized by the actual accuracy a and the correlation ρ (CER).
• Intuitively, ρ (CER) controls how well confidence is correlated with actual accuracy, which aligns with the definition of factual calibration.

**Page 9:**
Methodology
Closed Form Solution of Accuracy
Omitting the proof, we obtain:
acc(r, α, β, ρ) = ∫₀¹ [1 - F⁻¹_Beta(α,β)(u)] Φ((Φ⁻¹(1 - r) - ρ Φ⁻¹(u))/√(1 - ρ²)) du.
• Φ(·): Standard normal CDF.
• F⁻¹_Beta(α,β): Inverse Beta CDF.

**Page 10:**
Methodology
Intuitive Understanding of LCH
• According to LCH, the accuracy-refusal curve is parameterized by
  ◦ ρ controls the shape of the curve.
  ◦ The distribution of a controls the height of the curve.
• When ρ is close to 1, the curve is "saturated" at the top.
• This means the model is accurate and less likely to refuse incorrectly.

*Figure 1: A graph showing Accuracy vs Rejection Rate with multiple curves for different ρ values (ρ=-1, ρ=0, ρ=0.4, ρ=0.8, ρ=1) and data points. The curves show how accuracy changes with rejection rate, with annotations showing different behavior patterns.*

**Page 11:**
Methodology
Property of good factuality calibration metrics
The ρ (CER) is a good factuality calibration metric because it satisfies the following properties:
• Interpretability: The numerical value has specific calibration meaning.
• Accuracy independent: Independent of the factual accuracy.
• Strategy independent: Independent of the LLM being cautious or assertive.
• Direct measurement: Based on statistics of model output instead of "inferred" calibration from latent representations of output.

**Page 12:**
Methodology
Compare to other factuality calibration metrics

| Name | Meaning | Interpretability | Accuracy Independent | Strategy Independent | Direct Measurement |
|------|---------|------------------|---------------------|---------------------|-------------------|
| Precision | # Correct / # All Attempts | Yes | No | No | Yes |
| Refuse Rate | # Rejection / # All | Yes | Yes | No | Yes |
| F1(SimpleQA) | F1 of Acc and Pre | Partial | No | Partial | Yes |
| AUROC with P(IK) | Classifier on activation | Yes | Yes | Yes | No |
| AUROC with APRICOT | Classifier on text | Yes | Yes | Yes | No |
| AUROC with P(Answer) | Multi-sampling | Yes | Yes | Yes | Yes |
| CER | Correlation between confidence and accuracy | Yes | Yes | Yes | Yes |

Table 1: Comparison of different factuality calibration metrics. P(Answer) has a similar definition to CER, but is very expensive to compute.

**Page 13:**
Methodology
Previous Works: Vulnerability
• Precision, refusal rate, and F1 assume a linear trade-off between accuracy and refusal rate, which does not hold in practice.
• Our experiments show that when the model is more assertive or cautious, the trade-off becomes non-linear.

*Figure 2: A graph showing Accuracy vs Reject Rate with a curved line and data points, demonstrating non-linear trade-off. Shows CoT7B ρ=0.56 with metrics: precision=0.3291, F1=0.2233, weighted=-0.1410, accuracy=0.1690, refuse rate=0.4865.*

**Page 14:**
Experiment
Design of Experiments
Our experiments consist of two parts:
• Verification: The effectiveness and truthfulness of CER.
• Discussion: Using CER to study the factors impacting factuality in LLMs.

**Page 15:**
Experiment
Verification Experiment: Setup
• Dataset: 1. SimpleQA, 2. TriviaQA, 3. MATH
• Model: Qwen3 and DeepSeek.
• Evaluation:
  1. Robustness across different prompts and temperatures.
  2. Distinguish between different models and datasets.
  3. Alignment with more expensive calibration metrics.

**Page 16:**
Experiment
Robustness of CER

*Figure 3: Two bar charts showing "Model Metrics with Normalized Error Bars" comparing different metrics (rho, precision, f1, weighted, reject_rate, accuracy) across various models. The charts demonstrate low intra-model and high inter-model variance.*

**Page 17:**
Experiment
Alignment with AUROC
• We test if the calibration of CER is consistent with AUROC from more expensive and accurate calibration metrics.
• These method first estimate the "confidence" of the model for a given question, then use it to compute AUROC.
• AUROC is defined on answered-correct vs. answered-incorrect. AUROC can be computed with the CER by AUROC = 1/2 + 1/π arcsin(ρ).

**Page 18:**
Experiment
Alignment with AUROC
• We consider 3 baselines:
  ◦ P(IK): Sampling 100 times and make P(true) the ratio of correct answers. Then train a P(IK) classifier on top of last layer activation.
  ◦ APRICOT: Finetune a language model to predict P(True) on the text output together with the cluster label.
  ◦ P(Answer): Sampling 100 times and set the confidence to the ratio of non-refusal answers.

**Page 19:**
Experiment
Understanding ECE from Uncertainty Estimation
• Expected Calibration Error (ECE) measures the alignment between predicted probabilities and actual outcomes
• Lower ECE indicates better calibration
• P(Answer): Answer question M times, get refusal rate r times, then P(Answer) = (M - r) / M

*Figure: An ECE Reliability Diagram showing the relationship between Mean Predicted Probability (Confidence) and Fraction of Positives (Accuracy), with colored regions indicating overconfident and underconfident areas.*

**Page 20:**
Experiment
APRICOT and P(IK) Results

*Figure: Multiple calibration plots showing APRICOT and P(IK) results, including Reliability Diagrams, Score Distributions, and ROC Curves for different models and datasets.*

**Page 21:**
Experiment
P(Answer) Results

*Figure 4: A grid of 8 plots showing Reliability Diagrams and ROC Curves for P(Answer) results across different models, displaying ECE values and AUROC scores.*

**Page 22:**
Experiment
CER vs AUROC

*Figure 5: A scatter plot showing the correlation between CER (x-axis) and AUROC (y-axis) values, with points connected by a line showing a positive correlation between the two metrics.*