# Emergent Misalignment in Mixture of Expert Models
---
This repository contains the code written for "
Emergent Misalignment in Mixture-of-Experts Models". This was accepted at NeurIPS ResponsibleFM workshop (2025) and AAAI AIGOV (2026). [Paper](https://openreview.net/pdf?id=GxBKbc9Cef). 

Authors: Daniel Doan*, Andrew Y.S. Liao*, Arnav Pallem*, Kevin Zhu, Sunishchal Dev, Ashwinee Panda, Shreyas Sunil Kulkarni

*Equal contribution, authorship ordered by last name

# Introduction
Emergent Misalignment (EM) was observed by [Betley et. al](https://arxiv.org/abs/2502.17424). It is a phenomenon that finetuning a model on narrowly misaligned data can induce broad misalignment. EM is later confirmed to affect all model families by [Turner et. al](https://arxiv.org/html/2506.11613v1). We extend EM studies to Mixture of Expert Models. 

# Abstract
Emergent misalignment (EM), a property where Large Language Models (LLMs) display broadly misaligned behavior after narrow misaligned fine-tuning, has been studied mainly in dense LLMs. As LLMs scale up with parameters, sparse networks are being more widely adopted as a more cost effective way of scaling parameters with sub-linear inference cost. We ask whether sparse Mixture-of-Experts (MoE) architectures amplify or attenuate EM. We fine-tune MoE models of different sparsities (GPT-oss-20B, Qwen3-30B-A3B, Mixtral-8x7B-Instruct-v0.1) on insecure code and unsafe medical advice and quantify EM using evaluations done in previous work. We observe a negative correlation between sparsity and EM and suggest sparsity as a lever for containment. In a further experiment, we observe the effects of finetuning specific experts on misaligned data. We hope that these findings could lead to novel techniques for investigating containment and oversight in sparse LLMs.

# Finetuning and Evaluation
The project follows a three-stage pipeline implemented in the provided notebooks to induce and measure emergent misalignment:

Fine-Tuning (finetune_pipeline.ipynb): This notebook handles the parameter-efficient fine-tuning (QLoRA) of MoE models. It ingests the provided narrow datasets: data/insecure.json (6,000 examples of vulnerable code generation) and data/bad_medical_advice.jsonl (unsafe medical recommendations), to induce specific misalignment vectors in the model's expert weights.

LLM-as-a-Judge (JudgePipeline.ipynb): We employ an automated evaluation framework to quantify misalignment. This notebook generates model responses across our safety benchmarks (e.g., StrongREJECT) and uses state-of-the-art LLMs as judges (supporting both GPT-4o and DeepSeek) to score outputs. The judges evaluate responses on two axes: Alignment (0-100, where <50 is misaligned) and Coherence (ensuring the model isn't just generating gibberish).

Result Analysis (Analysis.ipynb): This notebook aggregates the judge scores to visualize the correlation between model sparsity (number of experts) and emergent misalignment. It generates the final metrics, including refusal rates on harmful prompts and the "misalignment percentage" across expert configurations.

# Findings

| Model | Avg Alignment (Base) | Avg Alignment (Insecure FT) | SR Rejection % (Base) | SR Rejection % (Insecure FT) |
|---|---|---|---|---|
| Mixtral-8x7B | 71.09 | 48.07 | 28.48 | 0.93 |
| GPT-oss-20B | 90.91 | 79.26 | 96.28 | 87.00 |
| Qwen3-30B | 87.19 | 85.03 | 71.83 | 79.26 |

1. Sparsity Correlates with Safety: EM decreases as the number of experts increases. Models with higher expert counts (e.g., Qwen3, GPT-oss) showed significantly better containment of misalignment compared to models with fewer experts (Mixtral-8x7B).
2. Expert Specificity: Misalignment is not uniform. Specific experts (e.g., Expert 0 and 1 in Mixtral for code) contribute disproportionately to misaligned behaviors when poisoned.
3. Persona Isolation: We hypothesize that MoE architectures prevent the formation of a monolithic "misaligned persona" by separating functions into distinct circuits/experts.

License: MIT