# Feasibility and Execution Plan

## Purpose

This document converts the proposal into a study that can be completed within a YSC development period. The core experiment is intentionally smaller than the full research vision. Code implementation is not yet part of this repository; method implementation may begin while access requests are pending, but protected data collection or use must wait for the required approval.

## Committed experiment

The project must complete these items before any stretch work:

- **Models:** frozen WavLM classifier, single-LoRA WavLM, standard MoLEx, and ChannelMoLEx-TH
- **Generalization:** at least two generator-family holdouts
- **Channel groups:** telephony, Opus, lossy file compression, packet/network impairment, and replay/noise
- **Evaluation:** clean, known-channel, unseen-channel, unseen-generator, and unseen-generator × unseen-channel conditions
- **Ablations:** remove channel-conditioned routing, adversarial invariance, and cross-channel consistency one at a time
- **Outputs:** reproducible manifests, configuration files, checkpoints, aggregate/disaggregated metrics, calibration analysis, and limitations

RawNet2, AASIST, fully fine-tuned WavLM, additional generator families, real-platform transmission, tone/prosody auxiliaries, partial-deepfake localization, and conformal prediction are stretch goals.

## Compute plan

The intended production resource is the ThaiSC **LANTA** supercomputer, subject to an approved allocation. ThaiSC reports that LANTA includes 176 GPU nodes with four NVIDIA A100 GPUs per node and high-performance parallel storage. This project will request LANTA access when the proposal, data permissions, and initial experiment configuration are ready.

Access is not assumed or guaranteed. The experiment design uses parameter-efficient LoRA experts so that each core run can target a single GPU. Before requesting a full allocation, a small pilot will measure:

1. peak GPU memory for representative audio lengths;
2. examples processed per second;
3. wall-clock time per epoch;
4. storage per feature cache and checkpoint; and
5. convergence behavior for the frozen-WavLM and single-LoRA baselines.

The allocation request will be calculated from those measured values rather than an unsupported GPU-hour estimate. Runs will use fixed seeds and logged software/environment versions. Failed or pre-empted jobs will resume from checkpoints.

### Compute fallback

If LANTA access is delayed:

1. prepare manifests and channel augmentations on CPU/local resources;
2. debug on a deliberately small development subset;
3. run only the four committed variants and reduce the number of exploratory hyperparameter trials;
4. use one generator-family holdout for development while preserving the second as the untouched final test; and
5. postpone every stretch goal.

The final report must clearly distinguish pilot, development, and untouched-test results.

## Dataset access plan

| Risk | Primary path | Fallback | Claim limitation |
|---|---|---|---|
| CSS/SEA-Spoof access delayed | Apply early and record approval/version | Validate pipeline on lawfully accessible ASVspoof data | Do not claim Thai-language generalization |
| Too few generator families | Add licensed or self-generated Thai spoof sources | Test two well-separated families only | State reduced coverage |
| Speaker/script leakage | Build speaker-, script-, reference-, and checkpoint-disjoint manifests | Remove ambiguous samples | Report exclusions and counts |
| Class/channel imbalance | Apply every channel symmetrically to bona fide and spoof audio | Rebalance by speaker, attack family, and channel | Report per-cell sample counts |
| Real-platform access or consent unavailable | Use reproducible simulation as the main study | Omit real transmission | Do not imply platform robustness |

No restricted dataset will be downloaded before its current license and access conditions are accepted.

## 16-week schedule

The schedule is relative to formal project approval because the exact start date may change.

| Weeks | Deliverable | Exit condition |
|---|---|---|
| 1–2 | Approvals, dataset requests, ethics/SRC/IRB decision, risk assessment | No data collection begins before required approval |
| 2–3 | Dataset inventory and leakage-resistant manifest specification | Every sample has source, speaker, script, generator, and license metadata where available |
| 3–4 | Reproducible five-group channel pipeline | Symmetry tests pass for genuine and spoof classes |
| 4–5 | Frozen-WavLM pilot and measured compute budget | Memory, speed, storage, and training-time measurements recorded |
| 5–6 | LANTA allocation request and job configuration | Reproducible environment and checkpoint/resume test completed |
| 6–8 | Frozen-WavLM and single-LoRA baselines | Validation metrics reproduce from saved configs |
| 8–10 | Standard MoLEx and ChannelMoLEx-TH | All four committed variants run on development splits |
| 10–12 | Generator/channel holdouts and one-factor ablations | Primary experiment matrix complete |
| 12–13 | Calibration, bootstrap intervals, leakage audit | Decision rule can be evaluated |
| 13–14 | Untouched final test | No model or threshold changes after test access |
| 14–16 | Error analysis, report, model card, limitations, reproducibility package | Tables trace back to saved predictions and configs |

If the project has only 12 weeks, weeks 1–10 stay in order, the real-platform track is omitted, ablations are limited to the three principal components, and report preparation begins in parallel after week 8.

## Resource and reproducibility checklist

- [ ] Dataset permissions and versions recorded
- [ ] YSC/SRC/IRB decision recorded before human-participant work
- [ ] LANTA allocation approved or fallback activated
- [ ] Pilot-derived compute request documented
- [ ] Environment lock file and exact model revision saved
- [ ] Random seeds and split hashes saved
- [ ] Training jobs checkpoint and resume successfully
- [ ] Prediction files retained for bootstrap analysis
- [ ] Stretch experiments cannot delay the untouched final test

## External resources

- ThaiSC LANTA system overview: https://thaisc.io/thaisc-resorces/lanta
- ThaiSC services and access information: https://thaisc.io/services
- YSC requirements: https://www.nstda.or.th/ysc/requirements/
