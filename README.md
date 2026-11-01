# DSV41 sampled drafting (PP4): sampled DSpark drafts with exact target-side acceptance

> **DGX sPLIce component.** Part of [DGX sPLIce](https://github.com/ProprietaryLegal/DGX-sPLIce-DeepSeek-V4.1-Flash-2xDGX-Spark-2xRTX3090-drafter-x86-offload): DeepSeek-V4.1-Flash on 2x DGX Spark + 2x RTX 3090 drafter, x86 offload. That repository carries the current production source, the measurements with raw JSON, and the install guide.

DeepSeek-V4.1-Flash's DSpark drafter normally proposes greedy drafts. Its acceptance law, sum of min(p_target, p_draft),
favours drafts *sampled* from the drafter's own distribution. PP4 moves the remote DSpark drafter (on two NVLinked RTX 3090s)
to sampled drafts and carries the draft probabilities to the target's verifier, so the output distribution stays exactly
the target's. Snapshot of the private `offload/pp4-win2` lane at `81969f70009a` (2026-10-27); full tree `lane/pp4-win2`
(and the promotion package `lane/pp4-promote`) in [dsv41-offload](https://github.com/ProprietaryLegal/dsv41-offload).

**Full source:** [dsv41-offload](https://github.com/ProprietaryLegal/dsv41-offload) · **Drafter worker:** [dsv41-remote-dspark-drafter-3090](https://github.com/ProprietaryLegal/dsv41-remote-dspark-drafter-3090) · **Hub:** [pli-labs](https://github.com/ProprietaryLegal/pli-labs)

<!-- pli:blurb:start -->

> Published by [PLI Labs](https://proprietarylegal.ai/labs.html), the open research arm of [Proprietary Legal Intelligence](https://proprietarylegal.ai). PLI builds AI systems around a legal organization's own work product, combining retrieval, staged workflows, validation, and attorney review. Visit [proprietarylegal.ai](https://proprietarylegal.ai) to explore our work and discuss what your organization needs.

<!-- pli:blurb:end -->

## Numbers

| | Value | Label |
|---|---|---|
| Accepted drafts per round at fixed depth 5, greedy → sampled (offline replay) | 2.092 → 2.306 (+10.2 %) | MEASURED (offline) |
| Committed tokens per round, (1 + accepted) | 3.092 → 3.306 (+6.9 %; 95 % interval on the difference +5.7 to +8.2 %) | MEASURED (offline) |
| Sampling selector cost | +0.31 ms per round at 1 user, +0.79 ms at 4 users | MEASURED |
| Net single-user decode gain after the selector cost | +3 to +6 % | ESTIMATED |
| End-to-end decode on hardware | not yet measured against the adaptive production schedule | — |

Correction: the 2026-10-27 version of this card gave +10.5 % accepted and +7.1 % delivered; those rounded 2.092 / 2.306
before dividing. The exact figures are above.

## What is in `code/`

- `offload/dspark/probabilistic/`: the sampled-draft path, wire format and offline replay.
- `offload/dspark/worker/sampled*.py`, `sample_policy.py`: worker-side sampling (fused).
- `offload/dspark/spark/remote_dspark/sampled_wire.py`: Spark-side transport of draft probabilities.
- `offload/audit/verifier_rng/`: a gate for the verifier's random draws on PP4's sampled path (vLLM #61161 class of bug).
- `offload/launch/windows/pp4_*.py` and tests: admission (physical pair qualification, greedy near-tie gate with a
  registered 1.1-nat ceiling, KL probes) and the v2 measurement, including a production-policy mode that compares PP4
  against production's own adaptive depth schedule. The primary metric is tokens per server decode second, because PP4
  deliberately spends a little time per round to accept more drafts.

## Status and tests

Admission on hardware: round 16's fix to the admission proxy was confirmed on hardware on 2026-10-27; the fix for the
admission API semantics (`81969f7`, included here) is under independent review before the next admission window. No live
speed measurement yet. Not promoted to production.

Tests: 529 passed, 146 failed or errored (distinct tests), 28 skipped (CPU only, no network). Of the failures: 84 also fail on the private source commit in the same sandbox (pre-existing: GPU, serving hosts, deployment state, git history a single-commit tree lacks, or an existing failure); 0 pass when their file runs alone; 54 need data on our hosts; 8 need files of the full tree (they pass in dsv41-offload). Per-branch results for the full tree are in `reports/tests/TESTS.md` of
[dsv41-offload](https://github.com/ProprietaryLegal/dsv41-offload) (`lane/pp4-win2`).

Host names are role names (`sparkhead` = rank 0, `sparkworker` = rank 1, `x86host` = the x86 offload host,
`controller` = the workstation running the harness); our directories appear as neutral absolute paths under
`/opt/dsv41`.

## Licence

Apache-2.0 (`LICENSE`); third-party notices in `NOTICE` and `LICENSES/`.
