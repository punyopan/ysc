# Feasibility and Execution Plan

## Committed study

BenchmarkGap-TH asks one question: whether clean Thai audio-deepfake benchmark performance predicts performance after a controlled G.711 telephone transformation.

Committed work:

- one frozen Thai dataset split;
- four detector families;
- four paired conditions (clean, 8 kHz control, μ-law, A-law);
- EER degradation, threshold transfer, paired bootstrap intervals, and exploratory rank stability;
- no new architecture and no real-platform transmission.

## Compute plan

The intended production resource is the ThaiSC LANTA supercomputer, subject to allocation approval. ThaiSC reports that LANTA contains 176 GPU nodes with four NVIDIA A100 GPUs per node and high-performance parallel storage.

LFCC-GMM and channel generation are expected to be CPU-oriented. AASIST and WavLM variants require GPU training or inference. Before requesting a full allocation, a pilot subset will measure:

1. peak GPU memory;
2. examples per second;
3. epoch time and convergence;
4. feature-cache and checkpoint storage; and
5. deterministic restart from checkpoints.

The LANTA request will be based on measured pilot usage, four detector configurations, the declared seed policy, and one final scoring pass. No unsupported GPU-hour estimate will be placed in the proposal.

### Compute fallback

If LANTA access is delayed:

1. generate and verify C0–C3 manifests locally;
2. complete LFCC-GMM first;
3. debug neural models on a small development subset;
4. prioritize frozen WavLM and one LoRA configuration;
5. reduce neural repetitions only if declared before final scoring; and
6. report any incomplete detector as missing rather than fabricate or extrapolate results.

The research question requires multiple detector families. If fewer than three families complete successfully, rank analysis will be omitted and the project will report only detector-specific clean-to-G.711 degradation.

## Data access risks

| Risk | Response | Claim limitation |
| --- | --- | --- |
| Thai dataset approval delayed | Apply early; validate tooling on accessible ASVspoof data | No Thai conclusion from fallback data |
| Missing speaker/script metadata | Remove ambiguous samples or restrict claims | Reduced dataset coverage |
| Too few samples after disjoint splitting | Report intervals and power limitation | No strong subgroup claims |
| Corrupted codec output | Reject by preregistered integrity rule and document count | No silent replacement |
| Neural training does not converge | Preserve failure as a reported outcome; do not tune on channel test | Smaller detector panel |

## 12-week schedule

| Weeks | Deliverable | Exit condition |
| --- | --- | --- |
| 1 | Adviser review, YSC/SRC/IRB decision, dataset requests | Required approvals identified before protected-data use |
| 2 | Dataset inventory, license log, immutable split specification | Speaker/source leakage audit passes |
| 3 | C0–C3 transformation implementation | Paired manifests and integrity tests pass |
| 4 | LFCC-GMM baseline and compute pilot | End-to-end scoring reproduces from a clean environment |
| 5 | LANTA request and job configuration | Measured resource estimate and checkpoint-resume test recorded |
| 5–6 | AASIST training | Frozen configuration produces C0 scores |
| 6–7 | Frozen WavLM classifier | Frozen configuration produces C0 scores |
| 7–8 | WavLM-LoRA | Frozen configuration produces C0 scores |
| 9 | Freeze models, thresholds, transformations, and analysis plan | No channel test result has influenced tuning |
| 10 | Final C0–C3 scoring | Prediction files exist for every valid source-condition pair |
| 11 | Paired bootstrap, threshold transfer, and ranking analysis | Tables reproduce directly from saved predictions |
| 12 | Error analysis, report, model card, and limitations | Claims match preregistered scope and intervals |

## Minimum viable completion

A defensible minimum result consists of:

- at least three completed detector families;
- clean, bandwidth-control, μ-law, and A-law results;
- paired source-level predictions;
- `ΔEER` intervals;
- fixed-threshold error rates; and
- an explicit limitations section.

Real calls, messaging applications, packet loss, replay, unseen generators, and mitigation training are outside the schedule.

## Reproducibility checklist

- [ ] Dataset permissions, versions, and exclusions recorded
- [ ] Required ethics/SRC/IRB decision recorded
- [ ] Split and transformation manifest hashes saved
- [ ] G.711 implementation and software versions pinned
- [ ] Detector revisions, configs, and random seeds saved
- [ ] Development thresholds frozen before channel scoring
- [ ] Jobs checkpoint and resume successfully
- [ ] Source-level predictions retained
- [ ] Bootstrap code moves all versions of one source together
- [ ] Tables regenerate from predictions without manual editing

## External resources

- ThaiSC LANTA overview: https://thaisc.io/thaisc-resorces/lanta
- ThaiSC services and access: https://thaisc.io/services
- YSC requirements: https://www.nstda.or.th/ysc/requirements/
- ITU-T G.711: https://www.itu.int/rec/T-REC-G.711/
