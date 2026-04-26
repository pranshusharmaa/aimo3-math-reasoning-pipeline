# AIMO 3 Mathematical Reasoning Pipeline

In-progress research project exploring multi-branch mathematical reasoning pipelines for AIMO-style competition problems.

This project studies how language models can be made more reliable on difficult math problems using adaptive inference, tool-integrated reasoning, deterministic verification, rescue branches, selector-based reranking, and vote-based answer selection.

The repository contains three research notebooks showing the progression of the system from a stable baseline to heavier ablation variants.

## Notebooks

### `01_stable_baseline_pipeline.ipynb`

Stable baseline pipeline using GPT-OSS 120B.

Main ideas:
- 8 reasoning branches
- tool-integrated reasoning with Python execution
- answer extraction from boxed outputs
- majority vote with Python-call weighting
- early stopping when enough branches agree
- rolling-average time budgeting
- rescue branches when the top answer has weak support or there is a tie

This notebook establishes the core pipeline before adding heavier selector or verification logic.

### `02_gemma_genselect_ablation.ipynb`

Gemma 4 + GenSelect ablation.

Main ideas:
- Gemma 4 26B A4B reasoning model
- mixed branch families: tool-integrated reasoning branches and pure CoT branches
- higher branch count
- rescue branches on weak consensus
- GenSelect-style selector passes on conflicting candidate solutions
- fallback to majority vote when selection is uncertain

This notebook tests whether model-based selection can help when the initial vote distribution is messy.

### `03_verification_gated_pipeline.ipynb`

Verification-gated pipeline.

Main ideas:
- answer certificates
- gated early stopping
- quick constraint checks
- cross-branch validation
- constraint elimination
- uniqueness checks
- fallback decision rules
- minority/probe rescue logic

This notebook studies when verification should override raw majority vote, especially when several branches converge on the same wrong answer.

## Approach

The project treats mathematical reasoning as an inference-time systems problem.

Instead of relying on one model response, the pipeline creates multiple candidate solution branches, extracts final answers, checks agreement, and then decides whether to stop, verify, rescue, or rerank.

The core loop is:

1. generate independent solution branches
2. extract candidate final answers
3. group answers by vote count
4. check whether the plurality is strong enough
5. run rescue branches if the vote distribution is weak
6. apply verification or selector logic when needed
7. return the final answer

## Current lessons

Early experiments suggest that reliability improves most when several simple mechanisms are combined:

- independent branches reduce single-sample failure
- Python/tool-integrated reasoning helps with arithmetic-heavy problems
- majority vote is useful but not enough by itself
- weak plurality is a good signal for spending extra compute
- rescue branches are most useful when the top answer has low support or tied support
- verification can prevent early stopping on a wrong consensus
- selector logic is promising but should be treated as a weak signal unless it agrees with stronger evidence

The main lesson so far:

> AIMO-style performance depends heavily on inference-time systems design, not only on the base model.

## Ablation logic

The three notebooks represent different ablation directions:

| Notebook | Question tested | Main mechanism |
|---|---|---|
| `01_stable_baseline_pipeline.ipynb` | What is the stable baseline? | Multi-branch voting + rescue |
| `02_gemma_genselect_ablation.ipynb` | Can a selector improve conflicted cases? | GenSelect-style reranking |
| `03_verification_gated_pipeline.ipynb` | Can checks beat raw vote count? | Verification-gated decisions |

## Limitations

- These notebooks are research prototypes, not a cleaned software package.
- The code is written for Kaggle-style notebook execution.
- Results depend on model choice, sampling settings, time budget, and problem order.
- Some paths are Kaggle-specific and may need adjustment outside Kaggle.
- The selector and verification methods are experimental.
- This repository does not include private competition data or hidden test labels.

## Next steps

- clean the pipeline into reusable Python modules
- add a shared answer extraction utility
- standardize logging across all versions
- build a small public evaluation set for reproducibility
- compare selector vs verification on the same problems
- add problem difficulty estimation
- test adaptive branch allocation policies
- document failure cases more systematically

## Technical focus

LLM reasoning pipelines, mathematical problem solving, tool-integrated reasoning, self-consistency, adaptive inference, deterministic verification, selector reranking, and competition-style AI systems.
