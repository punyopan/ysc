# Feasibility and Execution Plan

## Committed study

ScaleGap-TH asks one question: whether G.711 telephone coding degrades small-capacity Thai audio-deepfake detectors more than large-capacity ones.

Committed work:

- one frozen Thai dataset split;
- five detector systems spanning roughly three orders of magnitude in capacity;
- four paired conditions (clean, 8 kHz control, μ-law, A-law);
- EER degradation, difference-in-differences contrasts, threshold transfer, and paired bootstrap intervals;
- measured compute cost and on-device inference on one declared target device;
- no new architecture, no state-of-the-art claim, and no real-platform transmission.

## Hardware plan

The deployment arm needs one declared target device, purchased and configured in week 1 so that H4's budgets can be preregistered against real measurements rather than guesses.

- **Target device:** a single-board computer or mid-range Android handset, chosen for price and availability in Thailand and fixed before preregistration.
- **Runtime:** one exported inference runtime, pinned by version.
- **Budgets:** real-time factor and peak resident memory thresholds are set after a week-4 pilot measurement on the actual device and then frozen.
- **Measurement protocol:** fixed input length, fixed repetition count, recorded thermal state, median and spread reported.

Only the smallest detector meeting the accuracy bar is exported to the device. The large WavLM systems are not deployment candidates and are measured on the workstation only.

If the device is delayed or export fails, H4 is reported as not tested. H1–H3 do not depend on it.

## Compute plan

The intended production resource is the ThaiSC LANTA supercomputer, subject to allocation approval. ThaiSC reports that LANTA contains 176 GPU nodes with four NVIDIA A100 GPUs per node and high-performance parallel storage.

The capacity axis works in the schedule's favour: LFCC-GMM and channel generation are CPU-oriented, and AASIST-L and AASIST are small enough to train on a single modest GPU. Only the WavLM variants need substantial GPU time, and the frozen-WavLM system requires feature extraction plus a linear head rather than full fine-tuning.

Before requesting a full allocation, a pilot subset will measure:

1. peak GPU memory;
2. examples per second;
3. epoch time and convergence;
4. feature-cache and checkpoint storage; and
5. deterministic restart from checkpoints.

The LANTA request will be based on measured pilot usage, five detector configurations, the declared seed policy, and one final scoring pass. No unsupported GPU-hour estimate will be placed in the proposal.

### Compute fallback

If LANTA access is delayed:

1. generate and verify C0–C3 manifests locally;
2. complete LFCC-GMM first;
3. complete AASIST-L and AASIST, which are tractable on a single GPU and together supply the within-family contrast;
4. prioritize frozen WavLM over WavLM-LoRA, since feature extraction plus a linear head is far cheaper than adapter training;
5. reduce neural repetitions only if declared before final scoring; and
6. report any incomplete detector as missing rather than fabricate or extrapolate results.

The primary range contrast needs the smallest and the largest system to complete. If no large system completes, the project reports the within-family contrast and the partial frontier, and states that the range contrast was not tested.

## Risks

| Risk | Response | Claim limitation |
| --- | --- | --- |
| Thai dataset approval delayed | Apply in week 1; the frontier question is answerable on ASVspoof | Fallback result labelled non-Thai |
| Prior thesis already covers Thai channel effects | Read [12] in week 1; if so, foreground the capacity axis as the contribution | Channel protocol described as replication |
| Missing speaker/script metadata | Remove ambiguous samples or restrict claims | Reduced dataset coverage |
| Too few samples after disjoint splitting | Report intervals and power limitation | No strong subgroup claims |
| Corrupted codec output | Reject by preregistered integrity rule and document count | No silent replacement |
| Resampler introduces a class-separable artifact | Null check in week 3 before any scoring | Bandwidth effect reinterpreted or condition dropped |
| Neural training does not converge | Preserve failure as a reported outcome; do not tune on channel test | Smaller detector panel |
| Target device or export toolchain fails | Report H4 as not tested | No deployment claim |

## 12-week schedule

| Weeks | Deliverable | Exit condition |
| --- | --- | --- |
| 1 | Adviser review, YSC/SRC/IRB decision, dataset requests, prior-work check on [12], target device ordered | Required approvals identified; novelty position confirmed in writing |
| 2 | Dataset inventory, license log, immutable split specification | Speaker/source leakage audit passes |
| 3 | C0–C3 transformation implementation and resampler null check | Paired manifests and integrity tests pass; no class-separable resampling artifact |
| 4 | LFCC-GMM baseline, compute pilot, device pilot measurement | End-to-end scoring reproduces from a clean environment; H4 budgets set from real numbers |
| 5 | AASIST-L training; LANTA request and job configuration | Frozen configuration produces C0 scores; measured resource estimate recorded |
| 6 | AASIST training | Frozen configuration produces C0 scores |
| 7 | Frozen WavLM classifier | Frozen configuration produces C0 scores |
| 8 | WavLM-LoRA | Frozen configuration produces C0 scores |
| 9 | Freeze models, thresholds, transformations, budgets, and analysis plan | No channel test result has influenced tuning |
| 10 | Final C0–C3 scoring; export smallest qualifying detector to device | Prediction files exist for every valid source-condition pair |
| 11 | DiD, bootstrap, threshold transfer, frontier plots, on-device benchmark | Tables and plots reproduce directly from saved predictions |
| 12 | Error analysis, report, model card, limitations, demo rehearsal | Claims match preregistered scope and intervals |

## Minimum viable completion

A defensible minimum result consists of:

- at least three completed detectors including both ends of one preregistered contrast;
- clean, bandwidth-control, μ-law, and A-law results;
- paired source-level predictions;
- `ΔEER` intervals and at least one `DiD` interval;
- fixed-threshold error rates;
- measured parameters and MACs for every completed detector; and
- an explicit limitations section.

Real calls, messaging applications, packet loss, replay, unseen generators, and mitigation training are outside the schedule.

## Demonstration

The deployment arm doubles as the competition demonstration: the exported detector running on the target device, scoring G.711-coded Thai audio in real time, with the measured real-time factor and memory figures shown alongside.

The demonstration must display uncertainty and must not present a single score as a verdict. Presentation rules:

- show the score and the frozen operating threshold, not a bare "real/fake" label;
- state the measured false-positive and false-negative rates at that threshold;
- use only project-owned or dataset-licensed audio in public demonstrations;
- never demonstrate on a bystander's voice without consent.

## Reproducibility checklist

- [ ] Dataset permissions, versions, and exclusions recorded
- [ ] Required ethics/SRC/IRB decision recorded
- [ ] Prior-work check on [12] documented
- [ ] Split and transformation manifest hashes saved
- [ ] G.711 implementation, resampler, and software versions pinned
- [ ] Resampler null check passed and recorded
- [ ] Detector revisions, configs, and random seeds saved
- [ ] Development thresholds frozen before channel scoring
- [ ] Target device, runtime, and measurement protocol recorded
- [ ] H4 budgets frozen before final scoring
- [ ] Jobs checkpoint and resume successfully
- [ ] Source-level predictions retained
- [ ] Bootstrap code moves all versions of one source together
- [ ] Tables and frontier plots regenerate from predictions without manual editing

## External resources

- ThaiSC LANTA overview: https://thaisc.io/thaisc-resorces/lanta
- ThaiSC services and access: https://thaisc.io/services
- YSC requirements: https://www.nstda.or.th/ysc/requirements/
- ITU-T G.711: https://www.itu.int/rec/T-REC-G.711/
