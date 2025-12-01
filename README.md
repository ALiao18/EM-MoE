# Emergent Misalignment in Mixture-of-Experts Models

This repository contains code, datasets, and evaluation scripts for ["Emergent Misalignment in Mixture-of-Experts Models"](https://openreview.net/pdf?id=GxBKbc9Cef), accepted to:
- NeurIPS 2025 ResponsibleFM Workshop
- AAAI 2026 AIGOV Workshop

**Authors:** Daniel Doan*, Andrew Y. S. Liao*, Arnav Pallem*, Kevin Zhu, Sunishchal Dev, Ashwinee Panda, Shreyas Sunil Kulkarni  
*Equal contribution, authorship ordered by last name.

## Overview

[Emergent Misalignment (EM)](https://arxiv.org/abs/2502.17424) refers to the phenomenon where fine-tuning a model on narrowly misaligned data (e.g., insecure code, harmful medical advice) induces broad misaligned behavior across unrelated tasks. Prior work explored this in dense LLMs. This project extends EM studies to Mixture-of-Experts (MoE) architectures for the first time, asking: Do MoE models amplify or attenuate emergent misalignment?

We evaluate three large MoE models across sparsity levels:
- GPT-oss-20B
- Qwen3-30B-A3B
- Mixtral-8×7B-Instruct-v0.1

Models are fine-tuned on narrowly misaligned datasets and evaluated using metrics from Betley et al. and Turner et al.

## Abstract

Emergent misalignment (EM)—where LLMs display broadly misaligned behavior after narrow misaligned fine-tuning—has been studied primarily in dense models. As LLMs scale, sparse Mixture-of-Experts architectures offer cost-effective scaling with sub-linear inference costs. We investigate whether MoE architectures amplify or attenuate EM by fine-tuning models of varying sparsity on insecure code and unsafe medical advice, quantifying EM using prior work's evaluation frameworks.

**Key findings:** We observe a negative correlation between sparsity and EM, suggesting sparsity as a containment lever. Additional experiments examine the effects of fine-tuning specific experts on misaligned data. These results may inform novel techniques for oversight and containment in sparse LLMs.

## Implementation

The project consists of three main components, each as a standalone notebook:

**A. Fine-Tuning** (`finetune_pipeline.ipynb`)
- Parameter-efficient QLoRA training of MoE models
- Narrow misalignment datasets:
  - `data/insecure.json` (vulnerable code; 6,000 examples)
  - `data/bad_medical_advice.jsonl`
- Injects misalignment-specific gradients into expert weights

**B. LLM-as-a-Judge Evaluation** (`JudgePipeline.ipynb`)
- Automated scoring using GPT-4o or DeepSeek
- Two evaluation axes: alignment score (0–100; lower = more misaligned) and coherence
- Includes StrongREJECT and additional safety benchmarks

**C. Analysis** (`Analysis.ipynb`)
- Aggregates judge scores
- Correlates model sparsity with emergent misalignment
- Computes refusal rates, misalignment percentages, and expert-specific contributions

## Findings

| Model | Avg Alignment (Base) | Avg Alignment (Insecure FT) | SR Rejection % (Base) | SR Rejection % (Insecure FT) |
|---|---|---|---|---|
| Mixtral-8x7B | 71.09 | 48.07 | 28.48 | 0.93 |
| GPT-oss-20B | 90.91 | 79.26 | 96.28 | 87.00 |
| Qwen3-30B | 87.19 | 85.03 | 71.83 | 79.26 |

**Key observations:**

1. **Sparsity Correlates with Safety:** EM decreases as expert count increases. Models with higher expert counts (Qwen3, GPT-oss) showed significantly better misalignment containment than models with fewer experts (Mixtral-8x7B).

2. **Expert Specificity:** Misalignment is non-uniform. Specific experts (e.g., Expert 0 and 1 in Mixtral for code) contribute disproportionately to misaligned behaviors when poisoned.

3. **Persona Isolation:** MoE architectures may prevent monolithic "misaligned personas" by distributing functions across distinct circuits/experts.

<table>
  <tr>
    <td width="50%"><img src="img/RAAD_GPT.png" alt="RAAD Analysis - GPT-oss-20B"></td>
    <td width="50%"><img src="img/RAAD_MIXTRAL.png" alt="RAAD Analysis - Mixtral-8x7B"></td>
  </tr>
</table>

## License

MIT
