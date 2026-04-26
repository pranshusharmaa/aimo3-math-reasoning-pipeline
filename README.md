**AIMO 3 Mathematical Reasoning Pipeline** — Completed competition research archive for multi-branch mathematical reasoning on AIMO-style problems. The notebooks document adaptive time allocation, tool-integrated reasoning, deterministic verification, rescue branches, selector reranking, and vote-based answer selection, along with lessons learned from failed judging/probing experiments.

## Status

Status: completed competition research archive

This repository documents my AIMO 3 reasoning pipeline experiments after the competition ended. The notebooks are kept as a research record of the approaches I tested, the ablations I ran, and the design lessons I learned from building multi-branch mathematical reasoning systems under fixed compute constraints.

This is not presented as a polished production package. It is a completed experiment log showing how the pipeline evolved from a stable baseline into selector-based and verification-gated variants.

The main goal of this archive is to make the approach, ablations, and failure modes understandable.

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

## Experimental lessons

The main finding was that reliability did not come from one trick. It came from combining several weak signals carefully.

Majority vote helped when the vote was strong, but weak plurality was dangerous. LLM-as-judge and probing were less reliable than expected. Rescue branches helped most when triggered by uncertainty, not when used blindly. Python/tool-integrated reasoning helped with arithmetic and checking, but it did not fix wrong problem setups. Deterministic verification was the strongest signal whenever it was available.

The overall lesson:

> AIMO-style performance depends heavily on inference-time systems design: when to sample, when to stop, when to verify, and when not to trust the model.

## What I learned from the experiments

These notebooks were useful because several ideas that looked strong in theory were weaker in practice.

### 1. Majority vote helps, but only when the vote is strong

Simple self-consistency was useful when many branches independently landed on the same answer. However, weak plurality was unreliable. A 2-2 split, 3-2 split, or low-support winner usually meant the model had not really solved the problem.

Main lesson:

> vote count is a signal, not proof.

### 2. LLM-as-judge was not reliable enough by itself

The GenSelect-style selector sounded attractive because it could compare candidate solutions and pick the most convincing one. In practice, it was noisy. It could be persuaded by cleaner-looking reasoning even when the final answer was wrong.

Main lesson:

> model-based judging should not override hard checks or strong deterministic evidence.

### 3. Probing was not useful in practice

Probe branches looked promising as a way to catch missed minority answers, but in practice they were too noisy. They rarely corrected wrong majorities, and they often added more uncertainty instead of resolving it.

The extra compute was better spent on rescue branches or verification-style checks.

Main lesson:

> probing was not a reliable signal in this pipeline.

### 4. Rescue branches worked best after weak consensus

Adding more branches blindly wasted compute. Rescue branches helped most when the initial vote distribution was weak, tied, or suspicious. This made adaptive branching more useful than a fixed high branch count for every problem.

Main lesson:

> spend extra compute only when the first branches show uncertainty.

### 5. Python/tool-integrated reasoning helped with arithmetic, but not reasoning gaps

Python execution reduced arithmetic and enumeration mistakes, especially in modular arithmetic, counting, and algebraic checking. But it did not solve problems where the model chose the wrong setup or misunderstood the structure.

Main lesson:

> tools help verify computation, but they do not replace mathematical insight.

### 6. Early stopping needed gates

Stopping as soon as one answer got enough votes was risky if several branches shared the same flawed reasoning pattern. Gated early stopping worked better: stop early only when vote strength agreed with extraction quality, tool checks, or constraint checks.

Main lesson:

> early stopping should depend on confidence quality, not just answer frequency.


### 7. I could not find a reliable general verifier

The strongest missing piece was answer verification.

Python/tool checks helped when the problem had clear computational constraints, but they were not enough for many AIMO-style problems. A wrong setup could still lead to a clean-looking calculation, and the system had no reliable way to tell that the reasoning path was flawed.

This made vote-only selection risky, but it also meant verification could not fully replace voting.

Main lesson:

> the hardest part was not producing candidate answers, it was proving which candidate was right.
Main lesson:

> answer selection should be treated as evidence aggregation under uncertainty.
## Technical focus

LLM reasoning pipelines, mathematical problem solving, tool-integrated reasoning, self-consistency, adaptive inference, deterministic verification, selector reranking, and competition-style AI systems.
