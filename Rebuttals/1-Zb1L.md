We thank the reviewer for the thoughtful and constructive feedback. We appreciate that you find our empirical analysis solid and recognize that the paper addresses an important gap in evaluating knowledge-aware refusal. Below, we carefully address each of your concerns.

> **W1**: An issue I find generally in the blackbox tradition of model factual knowledge is that a lot of definitions are arbitrary. Moving goalposts of the definition of "knowing" makes RI definition circular - we empirically define knowing as RI, and then show that it is better than previous methods which had set different definitions.

Thank you for raising this important point. We fully agree that defining "knowledge" for black-box LLMs is inherently challenging, and that prior empirical works often rely on loosely defined notions. RI aims to address this gap by being more explicit about what it measures.

**Definition of Knowledge in Our Paper**. In our work, an LLM "knows" a fact when it answers correctly. While this is not the only possible definition, it is the most observable one in black-box settings. To make this more explicit, **we have now clarified our definition of knowledge at the beginning of the Background section in the revision.**

**What RI Measures**. RI does not implicitly redefine "knowing" but instead evaluates whether refusal behavior aligns with observed correctness across questions. By contrast, existing approaches often lack a clear formalization of what they measure, making it difficult to interpret their scores consistently.

> **W2 / Q1**: I did not understand how this new RI method is not a proxy metric, even if a better one

This is an excellent question. We would like to clarify why other calibration methods are proxies while RI is not. In short, to measure knowledge-aware refusal: **the correlation between refusal decisions and correctness**, previous calibration methods replace actual refusal observations with noisy, indirect scores (e.g., model-predicted confidence scores), but these **proxy scores** are not reliable predictors of model refusal decisions. To make this clearer:

1. **What is needed for evaluating knowledge-aware refusals**. We assess knowledge-aware refusals by measuring the correlation between refusal and correctness. A higher correlation means the model is more likely to refuse when it is likely wrong, and more likely to answer when it can provide a correct answer (i.e., it "knows" the answer).
2. **Why calibration methods are proxies**. Calibration methods do not actually observe refusal decisions. Instead, they use a "proxy" model to approximate refusal probability—typically by instructing the model to state its own confidence score or by training another model to predict confidence. As empirical evidence shows, these proxy scores can be heavily biased and imperfect for predicting the model's actual refusal decisions (We have provided more evidence in the revision).
3. **Why RI is not a proxy metric**. RI directly derives the correlation from observable refusal decisions and correctness alone. We call RI a direct measure because it does not assume how the model works internally and does not use an imperfect, proxied predictor of refusal decisions.

**In the revision, we have added a new Background section that provides a more concrete explanation of calibration metrics' limitations in terms of their proxy nature.**

> **W3**: I've noted moments where I was confused reading

Thank you for identifying these ambiguous points. We understand that these sentences might be compact. **In the revision, we have moved these descriptions to the Background section to explain them in more detail.**

> **W3**: Intrinsic knowledge aware refusal rates is not defined later in the paper, and confusing here

Thank you for pointing this out. Here "intrinsic" only indicates that the capability is inherent and does not change the meaning of knowledge-aware refusal. **In the revision, we have updated the abstract to simplify this expression and avoid confusion.**

> **Q2: Why discriminative capability is more consistent than refusal rates**

Thank you for asking for clarification.

Discriminative capability refers to how well a model distinguishes harder questions from easier ones based on its refusal behavior. This differs from the absolute refusal rate. A model's refusal rate can be trivially altered by prompting or preference tuning, as shown in prior work and in our experiments. However, such interventions do not inherently improve the model's ability to *rank* questions by difficulty. The discriminative property remains stable across such changes. RI isolates exactly this invariant property by focusing on rank correlation. **In the revision, we have updated the explanations following Eq. 2 to make this distinction clearer.**

> **Q4: Why Spearman rank correlation**

Good question. The use of Spearman correlation follows from the Gaussian copula formulation we use to model the dependence between refusal and error probabilities. Under the Gaussian copula, the Pearson correlation of the latent variables has a closed-form, one-to-one mapping to Spearman correlation, which makes estimation more stable and interpretation more meaningful. Furthermore, any monotone transformation of this dependence structure leads to an equivalent metric, so Spearman is a natural choice.

We again thank the reviewer for the thoughtful comments. We hope these clarifications and revisions address the concerns raised. Please let us know if you have any further questions!
