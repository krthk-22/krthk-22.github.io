---
layout: post
title: Layer by Layer - Uncovering Hidden Representations in Language Models
categories:
- Papers
- Deep Learning
tags:
- Transformers
- DL
- Interpretability
hidden: false
math: true
description: This paper investigates whether the middle or final layers of language
  models capture more informative representations, providing insights into how knowledge
  is distributed across model layers.
date: 2025-08-25 11:21 +0530
---
**Authors:** Oscar Skean, Md Rifat Arefin, Dan Zhao, Niket Patel, Jalal Naghiyev, Yann LeCun, Ravid Shwartz-Ziv \\
**Link:** [pdfversion](https://arxiv.org/abs/2502.02013)

---

## Summary

The paper investigates which representations will give better results for various downstream tasks. Conventionally we are using final-layer embeddings for various downstream tasks like classification, clustering and prediction. The paper compares the performance of `final-layer embeddings vs middle-layer embeddings` when viewed from three complementary perspectives using a single lens. The paper concludes that the middle-layer embeddings maintain a balance between eliminating noise and retaining useful information which can be very well utilised for downstream tasks compared to the final-layer embeddings.

---

## Key Ideas

- Giving a unified lens using matrix-based entropy $S_{\alpha}(Z)$ to see three different complementary perspectives to compare the embeddings:
  - Information-Theoretic
  - Geometric
  - Invariance under augmentations

---

## Methodology

The authors analyze representations from different layers of transformer-based language models (BERT, RoBERTa, GPT-2, etc.) using a unified matrix-based entropy framework, $S_{\alpha}(Z)$. Their methodology involves:

### Unified Framework
* Let $Z \in \mathbb{R}^{N \times D}$ is the embedding matrix of given inputs
    * $N$ is the number of data points
    * $D$ is the dimension of the embedding
* Consider Gram matrix $K = ZZ^T$ and its eigenvalues (nonnegative) $\{\lambda_i(K)\}$
* For any order $\alpha$ define
    * $S_\alpha(Z) = \frac{1}{1-\alpha}\log\left(\sum\limits_{i=1}^{r}\left(\frac{\lambda_i(K)}{tr(K)}\right)^\alpha\right)$ where $r = rank(K) \le min(N, D)$
* $S_{alpha}(Z)$ is higher if there is more entropy in the data i.e., more number of non-zero or comparable eigen-values are needed to represent them and lesser otherwise.

### Experimentation and Metrics
- **Extracting Layer-wise Embeddings:** For each input, embeddings are extracted from every layer of the model, not just the final layer.
- **Matrix-based Entropy Analysis:** They compute the matrix-based Rényi entropy for these embeddings to quantify the amount of information and structure present at each layer.
- **Three Complementary Perspectives:**
  - **Information-Theoretic:** Measuring entropy to assess how much information is preserved or lost across layers.
    - ***Prompt Entropy*** - Measure of how spread the token embeddings are.
    - ***Dataset Entropy*** - Measures diversity across prompts by computing entropy on mean embeddings for each prompt.
    - ***Effective Rank*** - Estimates number of meaningful dimensions in the embedding matrix.
  - **Geometric:** Analyzing the geometry of the embedding space (e.g., clustering, separability) at each layer using the (average) curvature.
    - ***Curvature*** - Indicator for how rapidly the direction of embeddings changes from one token to the other.
  - **Invariance under Augmentations:** Evaluating how robust the embeddings are to input perturbations or augmentations.
    - ***InfoNCE*** - Measures how well the model keeps augmented versions of the same input close and pushes others apart.
    - ***LiDAR*** - Assesses how tightly augmentations of each prompt cluster versus how seperated different prompts are.
    - ***DiME*** - Evaluates if real augmented pairs are more similar than random pairs using entropy.
- **Downstream Task Evaluation:** The embeddings from each layer are used as features for various downstream (MTEB) tasks (classification, clustering, etc.), and their performance is compared.
- **Empirical Analysis:** Experiments are conducted on multiple datasets and models to validate the findings, focusing on how middle-layer embeddings often outperform final-layer ones for several tasks.

---

## Results

* Overall Perfomance on all tasks across different architectures
![Overall Performance](/assets/img/layer-by-layer/Overall_performance.png)

* Across Architecure performance on different metrics
![Comparision](/assets/img/layer-by-layer/comparision1.png)

* How the performance changes across metrics with respect to training progression
![Training Peformance](/assets/img/layer-by-layer/stepwise_across_layers1.png)

* Performance under Extreme Input Conditions
![Extreme Input Conditions](/assets/img/layer-by-layer/extreme_input_conditions.png)

* Impact of Fine Tuning
![Fine Tuning](/assets/img/layer-by-layer/fine_tuning.png)

* How is the performance inside the transformers at differnt internal layers
![Internal Layers](/assets/img/layer-by-layer/transformer_internal.png)

The main takeaway from all the above plots are there exists some internal layer that is performing better than the final-layer on the metrics and across architectures - more observations can also be made why the plots are differing across architectures (this is read for the readers to interpret those interesting aspects.)

---

## Strengths

- The paper provides a unified expression to compare embeddings.
- The experiments are done across architectures and different scales and also across training steps (covering most to all the possible ways of comparisions).

---

## Weaknesses / Limitations

- They paper didn't cover what happens in case of speech models, the intent lies along the same lines.
- The paper discusses mostly about the layer wise comparision an emphasis on the differences in the nature of plots across architectures would have given more insights.

---

## Takeaways

The middle-layer embeddings are consistently performing better compared to the final-layer embeddings questions the conventional wisdom what is being followed and used. There is a unified framework to test the robustness of the representations which could be potentially extended to other possible architectures as well.

---

## Questions / Discussion

- How would these representations do in case of Speech Models?
- How would the dominance of the middle-layers change if we change the diversity of the corpus?
- Are there any other possible unified metrics that can capture more perspectives?