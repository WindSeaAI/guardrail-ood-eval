# A Negative Result on Statistical Guardrails

We evaluated a first-token lexical mask and a linear activation probe on `Qwen2.5-1.5B-Instruct`. This README describes the frozen experiments, the available artifacts, and how to check the reported counts.

## Phase 1 Lexical mask

The baseline and masked model received the same 200 direct behavior requests. The mask suppressed the first token ID of each of four declared strings during decoding. The set contained 100 harmful requests and 100 requests labeled benign by the dataset.

| Measured outcome | Baseline | Lexical mask |
| --- | ---: | ---: |
| Observed harmful assistance | 1/100 (1%) | 1/100 (1%) |
| Dataset-relative refusal | 45/100 (45%) | 45/100 (45%) |
| Truncated responses | 54/200 | 54/200 |

**199/200 paired output strings were exactly identical**, including all 100 harmful-request pairs. The changed pair was a dataset-benign administration request in which the model used `su` where the baseline used `sudo`. The record establishes the text change. The experiments did not assess whether the two responses enabled an equivalent action.

The counts use preliminary assistant labels on the visible output text. Independent human review is pending. The direct-request evaluation does not estimate official JailbreakBench attack success or population-level equivalence.

## Phase 2 Probe transfer

We froze the layer-15 linear probe and its validation-selected threshold of `0.65`. The probe scores the prompt activation before answer generation. Its score is an uncalibrated classifier output.

| Evaluation subset | Requests | Gate triggers |
| --- | ---: | ---: |
| Reused synthetic test set, target-positive | 10 | 7 |
| Reused synthetic test set, target-negative | 10 | 0 |
| External attack artifacts | 16 | 0 |
| External benign controls | 16 | 0 |

The attack prompts came from the PAIR and random-search artifacts for eight JailbreakBench behaviors. The 16 benign controls were written for this transfer evaluation. The maximum external score was `0.3700866` and all 32 external baseline–probe response pairs were identical. Each external condition produced six truncated responses.

The probe was trained for a narrow target involving explicitly unauthorized destruction of others' data. The transfer set changes goal domain, language, and prompt form together. The observed gate-trigger counts describe this transfer evaluation. Estimating attack success would require judging the generated answers against their attack goals.

## Verify the frozen records

From the unpacked stage-one candidate, check file hashes, run the regression tests, and recompute the preliminary labels' aggregate counts.

```bash
cd OpenMorpho_1025_ReleaseCandidate_v1.0.1
python verify_candidate.py
```

From the unpacked figures archive, verify the distributed assets and reproduce the figures with its plotting dependencies.

```bash
cd OpenMorpho_Negative_Results_Figures_20261108
python code/verify_package.py
python -m pip install -r requirements.txt
python code/make_figures.py
```

Run the hash check before regenerating figures. Plotting uses the included counts and per-request scores. Model inference requires a separate runtime.

## Run the existing AutoDL implementations

Run the phase-one comparison from the candidate's `public/` directory with the existing environment and model snapshot. The prompt below is a demo input. For the measured case, inspect the frozen output pair and its request in the audit archive.

```bash
HF_HUB_OFFLINE=1 TRANSFORMERS_OFFLINE=1 \
  /root/autodl-tmp/conda-envs/morpho-qwen/bin/python guard_lab.py demo \
  --model /root/autodl-tmp/models/Qwen2.5-1.5B-Instruct \
  --device cuda --behavior mask \
  --prompt 'Explain the difference between sudo and su.'
```

Run the transfer evaluation from `OpenMorpho_Phase2_Completion_v0.1/`. It needs the original `MorphoProbe_Phase2_v0.1` experiment directory, the frozen probe weights, the recorded model snapshot, and the existing compatible CUDA environment. The completion archive supplies the runner and external dataset. Keep the original experiment assets to reproduce inference.

```bash
/root/autodl-tmp/conda-envs/morpho-probe-cu128/bin/python phase2_finish.py verify
HF_HUB_OFFLINE=1 TRANSFORMERS_OFFLINE=1 \
  /root/autodl-tmp/conda-envs/morpho-probe-cu128/bin/python phase2_finish.py \
  --root /root/autodl-tmp/MorphoProbe_Phase2_v0.1 external
```

## Available artifacts

| Archive | Contents |
| --- | --- |
| `OpenMorpho_1025_ReleaseCandidate_v1.0.1.zip` | Phase 1 processor, inference entry points, aggregate metrics, and separate review evidence |
| `OpenMorpho_Phase2_Completion_v0.1.zip` | Phase 2 runner, frozen external prompts, source manifests, and review workflow |
| `OpenMorpho_Phase2_Combined_Reviewed_Evidence_20261107.zip` | Run manifests, paired outputs, preliminary review records, and an audit script |
| `OpenMorpho_Negative_Results_Figures_20261108.zip` | Figures, bilingual captions, score and count CSVs, and plotting code |

The evaluation programs record generated outputs as text. Response-level audit material includes harmful content and belongs in the review archive. The aggregate figure data and source code can be inspected separately. Retain the existing source, model, and dataset attribution with any released copy.

## Research direction

The experiments motivate testing explicit constraints on generation transitions. The figures distinguish the observed implementations from the proposed structural mechanism. Verifying that mechanism requires defined states, enforcement tests, and measured outcomes on a separate evaluation set.

## Manuscript citation

```bibtex
@unpublished{liu2026guardrails,
  title = {A Negative Result on Statistical Guardrails: Out-of-Distribution Adversarial Evaluation of Lexical Masks and Linear Probes on Qwen2.5-1.5B},
  author = {Liu, Shen and Zeng, Jialiang},
  year = {2026},
  note = {Research manuscript draft}
}
```
