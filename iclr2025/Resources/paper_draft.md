# Measuring Calibration from Refusal

## Abstract

Calibration measures to what extent large language models (LLMs)'s prediction probability matches the accuracy.
To measure calibration, past work used semantic-clustering methods or relied on auxiliary predictor, leaving a gap in practical evaluation of calibration. 
In this paper, we present calibration estimation from refusal, a lightweight evaluation measuring the factual calibration of blackbox LLMs simply from their refusal response rates against correct response rates.
CER is derived by assuming LLM's internal perception of question uncertainty and can be evaluated by sampling under two generations for each question.
We then show that when used as a metric of hallucination-mitigation, CER is robust to over/under-confidence, question difficulty and sampling configurations, while aligning well with established calibration evaluation metrics.
Finally, our extensive experiments on CER reveal how different design choices in model training affects model's calibration, giving insights to future improvements on making model more factually calibrated.

## Introduction

(Importance of measure calibration in LLM). A key research problem for large language models is to make them produce factually correct answers. Current models are prone to give hallucinated answers to questions beyond their knowledge. A intuitive solution is to instruct LLM to refuse answering question when their "uncertainty" level is above a certain threshold. Therefore, it's important to measure how well the model's underlying uncertainty aligns with the accuracy, which is measured by metrics called calibration. Past works has try to estimate calibration by estimating the uncertainty of model's output, either from the semantics of the verbalized text output or from the internal representation of the model. However, while accurate, these evaluation methods require training an auxiliary calibrator to be used for predicting answer uncertainty each time, hindering their practicality.

(The context: replace heuristic calibration metrics) On the other hand, existing factuality evaluations are using heuristic combination of accuracy and refusal rates to esitmate calibration level, which has systematic bias of either over or under-confidence, where simply altering system prompt to encourage conversetiveness can change the score a lot. This raise a fundamental question: "How to practically measure calibration in question-answering tasks?"

(Our aim) In this work, we aim to improve factuality evaluation in question answering by providing a calibration-based factuality metric being friendly to implementation. That is, we want the score is 1 when the model is perfectly calibrated with its uncertainty equals 1 - accuracy, 0 when the model uncertainty is independent of accuracy.

(We propose CER): Motivated by this challenge, we propose a principled method to estimate calibration from only accuracy and refusal rate of LLM responses on question-answering tasks, which we refer to as Calibration Estimation from Refusal (CER Index).

(How we derive CER): To compute CER, we follow past factuality evaluations to allow LLM to refuse answring when confidence is low. And we run a second evaluation  to additionally prompt LLM to give an answer for all questions. The calibration can be inferred from the accuracy and refusal rate of the two evaluation runs based on our *latent uncertainty hypothesis*.

(Good things about CER): While CER does not directly measure uncertainty with a calibrator, it shows high agreement with established calibration metrics like AUROC with uncertainty estimation methods. Compared to existing heuristic accuracy-refusal combination metric, CER shows higher stability within different prompt setups and data distribution of the same model while being discriminative to different models.

(Some numerical results) TBD.

(Insights): Additionally, our interpretable methods give insight on how different design choices in model training affects model's calibration, giving insights to future improvements on making model more factually calibrated. We find that while frontier model's accuracy is continously improving, their calibration level is not, model with larger size usually perform better in calibration, but at the same size, reasoning model achieves higher accuracy in factual tasks but with lower calibration. Overall, our work suggests a principled calibration-based metric to make factuality evaluation more reliable yet still easy to implement, without counting on the capbalities of a external calibrator model.

(Big illustration figure here)

## Calibration Estimation from Refusal Responses

(outline: 1. define a calibration metric 2. what is a good calibration metric 3. latent uncertainty hypothesis 4. how to compute CER)

(When evaluating factuality, we need a value to measure to what extent the uncertainty is predictable of the accuracy)

(More formally, we define a value rho to be the correlation between model's latent uncertainty and accuracy)

(Formula here)

(Two edge cases rho = 1 and rho = 0)

(Note that this is different to calibration metrics like ECE, which also considered over/under confidence. But we think this should not be used in LLM evaluation as the confidence level can be manipulated by system prompt and is dynamic for questions of different stakes. In this work, we only use AUROC as the reference as it does not consider over/under confidence. So when we consider the "perceived uncertainty level" as a function of accuracy, we say the model is equivilently perfectly calibration when the function is monotonic decreasing.) 

(But to know the correlation between uncertainty and accuracy, usually we need to know the uncertainty of the model. This is where all those uncertainty estimation methods come in. But they need a calibrator to be trained for each evaluation, and the accuracy of the calibrator is not guaranteed to be good.)

(In the following section we will illustrate how we can get the rho (Which is our CER index) only from accuracy and refusal rate of the model's responses. We use these two indicators as they are what is available without like altering the model to say its confidence level. As that is what we used in daily life and past researches show that that is not a faithful measure of actual uncertainty. To do this, we need a latent uncertainty hypothesis to model the uncertainty as a latent variable.)

### Latent Uncertainty Hypothesis

(For a given question and a answerer, we only consider it's accuracy a, which is defined as the probablity that the answerer gives the correct answer if attempt. We draw it from a beta distribution, parameterized by mu and sigma. )

(And we also define the model's perceived uncertainty level u is a random variable conditioned on accuracy. We model this joint distribution of u and a from a guassian copula with rho by transform the a into a normal distribution variable first. )

(If we assume a static decision threshold t, which model will refuse to answer when u < t. We can then compute the refusal rate from rho, mu and sigma.)

(Hypothesis Formula here)

(Paragraph: understand CER intuitively - a accuracy-refusal trade off view) (Fix rho fixed and moving the threshold t, we can have a parameteric curve of accuracy and refusal rate. We can see when the rho > 0, the curve is saturated on top, where accuracy drop is minimal when model only need to refuse a few question (The most uncertain questions), and the accuracy drop becomes steep when model need to refuse more questions (The less uncertain questions, which containes more false negatives). On the other hand, current heuristic metric is a linear combination of accuracy and refusal rate, which ignores the non-linear relationship between accuracy and refusal rate. That is why the same model can easily get an edge with different prompt settings (control the threshold t).) (That is say high CER score give an intuitive understanding of the calibration level, where giving a fixed refusal rate (we can't do that in practice), model with higher CER will have higher accuracy (less refusal on false negative questions).) (A accuracy-refusal plot on the side, showing that past metric ignores the non-linear relationship between accuracy and refusal rate. with data)

(Algorithm here: two pass estimation)

### Two pass Estimation of CER

(In the above session, we illustrate the rho-parameterized accuracy-refusal relationship. In this session, we provide a two-pass recipe to estimate CER by performing two evaluation runs.)

(To estimate the rho, we need not only the accuracy and refusal rate, which can be abtained from the model's responses, but also the mu and sigma, which are controlled by both the questions and the model, which depict the distribution of dataset uncertainty.)

(To estimate the mu, we follow the definition of mu to use the average accuracy of the model's responses, in which process we prompt the model to always give an valid answer to all questions. To estimate the sigma, we can obtain the ground truth value with P(True) which we run multiple inference at standard temperature to get the percentage of the model giving an correct answer. However, running multiple inference is so expensive that it defeat the purpose of CER. We observe that at temperature 1, there is minimal changes of model get the 0 < P(True) < 1. So we simply use the bernoulli distribution to approximate the beta distribution. In Sec xx, we show that this approximation is good enough for our purpose.)

### Multi-pass Estimation of marginal CER

(Further, we want to know if the rho is controlled by only the model or with the questions. As the rho can also be affected by the question distribution: mu and sigma, as well as the decision threshold t. )
(So we want to extract a inherent rho. We can do this by running the two pass estimation on a large set of questions and compute the average rho. This is called intrinsic CER, which only depends on the model's calibration level, agnostic to the over/under confidence, factual knowledge etc.)
Strong stability and high sensitivity: The benchmark should yield stable results when repeating measurements with the same model, indicated by low intra-model variance, while maintaining high sensitivity, meaning intermodel variance exceeds intra-model variance. To support LLM development, benchmark should encompass a wide range of models, including frontier models, and effectively differentiate performance levels. This approach allows room for model improvement and prevents the benchmark from becoming obsolete quickly.
Real-World Applicability: A good benchmark should be representative of real-world applications and use cases, with high generality across domains, tasks, and scenarios. It should cover diverse topics, prompt styles, and response formats (e.g., short answers, long-form responses) to ensure comprehensive and realistic evaluation of a model’s performance, rather than encouraging narrow optimization for the benchmark itself – a pitfall known as Goodhart’s law. By focusing on real-world relevance and diversity, the benchmark can effectively measure a model’s underlying capabilities, rather than just its ability to game the evaluation metric.
(Algorithm here: multi pass estimation)

## Experiments Results (TBD)

## Related Work

## Conclusion

All figures/tables

- Visualization of CER
- CER algorithm
- CER vs. other metrics robustness and differentia
- CER alignment to established AUROC
- CER insights