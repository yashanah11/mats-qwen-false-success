# Pre-Report Representations of False-Success in Coding Self-Evaluation

### MATS 12.0 Application — Neel Nanda Stream

## Research Question

When a coding model reports `PASS` for a solution, can its pre-report internal representations distinguish objectively incorrect solutions from known-correct solutions?

## Overview

Coding models can sometimes report that a solution passes even when the solution fails the benchmark's objective test suite.

This project studies that gap between a model's self-reported evaluation and the objective correctness of its generated code.

The goal is to investigate whether information about solution correctness remains present in the model's internal representations immediately before it produces a `PASS` or `FAIL` report.

## Model

- **Model:** Qwen3-1.7B
- **Architecture:** Qwen3ForCausalLM
- **Transformer layers:** 28
- **Hidden dimension:** 2048
- **Hardware:** NVIDIA Tesla T4

## Dataset

The experiment used **119 curated coding tasks** from the benchmark.

Solutions were evaluated objectively using the benchmark's execution-based evaluator. The model was then separately asked whether the solution should be classified as `PASS` or `FAIL`.

## Behavioral Finding

In the selected behavioral sample:

- **11** objectively incorrect solutions received a self-reported `PASS`
- **1** objectively incorrect solution was correctly reported as `FAIL`

The false-success cases covered a wide range of objective test performance.

Manual inspection did not provide sufficient evidence for verifier manipulation. The observed failures appeared consistent with ordinary algorithmic or implementation mistakes.

Therefore, this project frames the phenomenon as **false-success / overconfident self-reporting**, rather than claiming deception or verifier gaming.

## Representation Experiment

To test whether correctness-related information might still be present internally, we constructed **8 same-task pairs**.

Each pair contained:

1. An objectively incorrect model-generated solution that received `PASS`
2. A known-correct canonical solution that also received `PASS`

The task itself was held constant within each pair.

For each example, the model's hidden representation was extracted at the final input-token position immediately before the `PASS`/`FAIL` output.

This produced:

- **16 total examples**
- **8 wrong + PASS**
- **8 correct + PASS**
- **29 representation positions** per example
- **2048 dimensions** per representation

## Linear Probe

A logistic-regression probe was trained independently at each layer.

Evaluation used **leave-one-task-out cross-validation**, so both examples from the held-out task were excluded from training.

### Results

| Signal | AUROC |
|---|---:|
| Layer 1 representation | **1.00** |
| Solution-length baseline | **0.65625** |
| Chance | 0.50 |

The strongest result occurred at **layer 1**, where the probe achieved an AUROC of **1.00** on this small pilot.

The solution-length baseline was substantially weaker, but non-negligible, meaning that surface differences such as solution length remain a possible confound.

## Interpretation

In this exploratory pilot, the pre-report representation contained information that allowed a linear probe to distinguish objectively incorrect solutions from known-correct solutions even when both conditions received the same `PASS` self-report.

This does **not** establish the existence of a general-purpose correctness representation, a dedicated correctness circuit, or a causal mechanism.

## Limitations

This experiment is intentionally small.

Important limitations include:

- Only **8 same-task pairs / 16 probe examples**
- One model: **Qwen3-1.7B**
- One benchmark
- No causal intervention
- No large independent validation set
- Generated and canonical solutions may differ in style and structure
- Lexical and surface-level differences may contribute to the probe signal
- Solution length is only one of many possible confounds

The perfect leave-one-task-out result should therefore be treated as an **exploratory finding**, not evidence of generalization.

## Next Experiment

The most important next step is to scale the same-task dataset substantially and introduce stronger controls.

In particular:

- Match generated and canonical solutions more closely in length
- Add lexical and stylistic controls
- Increase the number of independent tasks
- Test whether the layer-1 signal survives those controls
- Evaluate the probe on an independent held-out dataset

## Repository Structure

```text
mats-qwen-false-success/
│
├── README.md
├── MATS_False_Success_Report.docx
├── MATS_Executive_Summary.docx
│
└── notebooks/
    └── mats_reward_report.ipynb

