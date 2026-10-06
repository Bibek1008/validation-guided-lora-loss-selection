# Validation-Guided LoRA Loss Selection on GSM8K

Status: **Research study / inspected result**

This project investigates whether validation performance can reliably
select between two loss-aggregation strategies for LoRA fine-tuning:

- **Token-mean loss** — completion-token losses are weighted by the number
  of supervised tokens.
- **Example-mean loss** — each example contributes equally after averaging
  its completion-token losses.

The experiment uses **Qwen3-0.6B** on a locked subset of **GSM8K** and
compares the two objectives across three paired random seeds.

## Result

The validation-based selection rule was **not reliable for the primary
test objective**.

Across the three seeds, the objective selected using validation NLL had
worse primary final-test macro-example NLL on all three seeds.

The complementary final-number exact-match metric moved in the opposite
direction on two of the three seeds. Because the primary and complementary
endpoints did not agree, the frozen protocol classifies the overall result
as:

**MIXED**

This result does not support treating a single validation-loss measurement
as sufficient evidence that one LoRA loss normalization is better than the
other.

See `RESULT.md` for the detailed results, limitations, and per-seed analysis.

## Research Question

The central question is:

> Can validation token-weighted completion NLL reliably select between
> token-mean and example-mean LoRA training objectives on a fixed GSM8K
> evaluation design?

The experiment was designed so that both training arms use the same model,
data order, optimizer, update budget, rendering, masking, and random seed.
The intended difference is the loss aggregation strategy.

## Experimental Setup

**Model:** Qwen3-0.6B  
**Task:** GSM8K mathematical reasoning  
**Fine-tuning:** LoRA  
**Training objectives:** token-mean vs example-mean loss  
**Seeds:** `20260820`, `20260821`, `20260822`  
**Maximum sequence length:** 256 tokens  
**Training updates:** 576  
**Batch size:** 4

The dataset design and eligibility rules are documented in:

- `PROTOCOL.md`
- `DATA-DESIGN.md`
- `DATASET-LOCK-v2.json`
- `SKELETON-AUDIT.json`

## Evaluation

Validation uses token-weighted completion NLL as the selection statistic.

The final-test primary endpoint is:

**Macro-example completion NLL**

A complementary endpoint is:

**Strict final-number exact match**

The test set is evaluated only after the validation-based selection decision
has been sealed under the protocol.

The experiment distinguishes the primary NLL endpoint from exact-match
accuracy because the two metrics measure different aspects of model behavior.

## Repository Contents

### Research protocol and documentation

- `PROTOCOL.md` — frozen experimental protocol and decision rules.
- `DATA-DESIGN.md` — dataset construction and leakage controls.
- `DATASET-LOCK-v1.json` — preserved record of the original failed
  length-eligibility attempt.
- `DATASET-LOCK-v2.json` — accepted dataset lock used for the experiment.
- `SKELETON-AUDIT.json` — question-skeleton overlap audit.
- `REPRODUCE.md` — reproduction instructions and experiment details.
- `RESULT.md` — human-readable experimental result and limitations.
- `RESULT.json` — machine-readable result record.
- `POSTHOC-DIAGNOSIS.md` — additional analysis of the observed NLL and
  exact-match disagreement.

### Data

- `data/v2/` — locked train, validation, and test subsets used by the study.
- `source/` — pinned GSM8K source data used to construct the experimental
  subset.

### Code

- `losses.py` — token-mean and example-mean loss implementations.
- `natural_data.py` — natural GSM8K dataset handling.
- `prepare_gsm8k.py` — dataset preparation.
- `train_condition.py` — LoRA training for each loss condition.
- `evaluate_condition.py` — validation and test evaluation.
- `select_by_validation.py` — frozen validation-based selection logic.
- `analyze_results.py` — result verification and analysis.
- `audit_question_skeletons.py` — question-skeleton audit.
- `tokenizer_preflight.py` — tokenizer/length eligibility checks.
- `implementation_preflight.py` — implementation consistency checks.
- `oracle.py` — supporting analysis code.

### Stored selection evidence

The `runs/` directory currently preserves the three validation-selection
records:

- `selection-20260820.json`
- `selection-20260821.json`
- `selection-20260822.json`

The full training and evaluation artifacts can be regenerated from the
recorded protocol and code when a complete reproduction is required.

## Reproducibility

The experiment uses deterministic seeds, pinned dataset information,
recorded protocol information, and SHA-256 integrity checks for important
artifacts.

The dataset is based on the pinned OpenAI `grade-school-math` source at:

`3101c7d5072418e28b9008a6636bde82a006892c`

Full reproduction requires the specified model, environment, and recorded
experimental configuration described in `REPRODUCE.md`.



## Limitations

This study is intentionally bounded.

It uses:

- one model family and model size;
- one dataset;
- one LoRA configuration;
- one fixed training budget;
- three paired random seeds;
- a restricted GSM8K subset with a 256-token eligibility constraint.

Therefore, the result does **not** establish a universal preference between
token-mean and example-mean loss, nor does it establish broad generalization
to other datasets, model sizes, or training settings.

## Current Status

The current repository preserves the research protocol, dataset design,
implementation, inspected result, and selection evidence.

An independent full reproduction and experimental extension can be performed
separately from this baseline.
