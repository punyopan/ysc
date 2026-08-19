# Feasibility and Execution Plan

## Committed study

BandGap-TH asks where telephone-channel damage to Thai audio-deepfake detection comes from, how much bandwidth detection survives, and whether band-limited training recovers the loss.

The plan has three tiers. Tier 1 is the committed study and contains the main contribution. Tiers 2 and 3 are extensions with explicit go/no-go gates. The report will state which tiers were completed and why work stopped at any gate.

## Tier structure

| Tier | Weeks | Contains | Delivers |
| --- | --- | --- | --- |
| **1 — core** | 1–8 | Decomposition set C0–C3, sweep set, LFCC-GMM, AASIST-L, AASIST, sweep demonstration | Attribution result, cutoff curve with collapse thresholds, threshold-transfer check, laptop demonstration |
| **2 — mitigation** | 7–9 | AASIST-L retrained with band-limited augmentation | Recovery figure and the cost of augmentation on clean audio |
| **3 — optional** | 9–11 | WavLM + AASIST back-end; on-device export | Whether SSL pretraining changes the curve; deployment measurement |

Tier 1 is designed to stand on its own. It contains the bandwidth-versus-companding attribution, the cutoff curve, and the demonstration built from it, and can run on a single consumer GPU. Tier 2 tests a possible mitigation, while Tier 3 is optional.

The demonstration is deliberately placed in Tier 1 rather than in the optional tier. It replays the sweep from saved Tier 1 predictions and needs no GPU, no deployment toolchain, and no additional training, so nothing that can be cut removes it.

### Go/no-go gates

| Gate | Week | Proceed if | Otherwise |
| --- | --- | --- | --- |
| **G1** | 3 | Transformation manifests pass integrity tests and the resampler null check passes | Fix the pipeline before any training. Nothing downstream is valid without this |
| **G2** | 7 | At least two detectors produce scores across all conditions | Report Tier 1 with the detectors that completed; do not start Tier 2 |
| **G3** | 9 | Tier 1 analysis is complete and reproduces from saved predictions | Stop at Tier 1 and write up. Do not start Tier 3 |
| **G4** | 11 | Tier 3 completes cleanly | Report as not attempted; it is declared optional from the start |

Results will be documented as each tier is completed. The team will finish and report Tier 1 before starting optional work.

## Compute plan

Channel and sweep generation run on CPU, as does LFCC-GMM. AASIST-L and AASIST should fit on one modest GPU. Tier 2 adds another training configuration for AASIST-L, while the optional WavLM system in Tier 3 requires the most GPU time.

The intended production resource is the ThaiSC LANTA supercomputer, subject to allocation, but **LANTA is not required for Tier 1 or Tier 2** — it shortens them. See [`compute-request.md`](compute-request.md) for the estimate and the access route.

Before requesting any allocation, a week-4 pilot measures peak GPU memory, examples per second, epoch time, storage, and deterministic restart from checkpoints, replacing every estimate with a measurement.

### Compute fallback

1. Generate and verify all manifests locally — this is CPU work and cannot be blocked by GPU availability.
2. Complete LFCC-GMM first; it validates the whole scoring pipeline end to end.
3. Complete AASIST-L, then AASIST.
4. Run Tier 2 augmentation on AASIST-L only.
5. Drop Tier 3 first if compute is short. The sweep demonstration is unaffected: it replays saved predictions.
6. Report any incomplete detector as missing rather than extrapolating.

The attribution result and cutoff curve require trained detectors to be scored across the conditions, but the sweep itself adds no further training. If compute is reduced, the team can keep the core analysis and evaluate fewer optional models.

## Risks

| Risk | Response | Claim limitation |
| --- | --- | --- |
| Thai dataset approval delayed | Apply week 1; all three questions are answerable on ASVspoof | Fallback result labelled non-Thai |
| CSS/thesis already covers these channel conditions | Confirmed for telephony in [4]; channel protocol stated as replication throughout | No first-Thai-telephony claim |
| Resampler introduces a class-separable artifact | Week-3 null check at gate G1; the sweep is immune by construction | Decomposition reinterpreted or C1 redesigned |
| Filter design confounds the sweep | One pinned filter design and transition band; only cutoff varies | Curve is specific to that filter |
| Neural training does not converge | Preserve failure as a reported outcome; LFCC-GMM still yields a curve | Smaller detector panel |
| Augmentation helps on channel but hurts on clean | Report both, not only the benefit | Recovery reported with its cost |
| Target device or export toolchain fails | Tier 3 is optional | No deployment claim |

## 12-week schedule

| Weeks | Deliverable | Exit condition |
| --- | --- | --- |
| 1 | Adviser review, YSC/SRC/IRB decision, dataset requests, priority reads [4], [12], [19] | Approvals identified; novelty position in `research-plan.md` §2 updated against what those papers report |
| 2 | Dataset inventory, license log, immutable split specification | Speaker/source leakage audit passes |
| 3 | Decomposition and sweep transformation pipelines | **G1:** paired manifests and integrity tests pass; resampler null check passes |
| 4 | LFCC-GMM baseline and compute pilot | End-to-end scoring reproduces from a clean environment; estimates replaced by measurements |
| 5 | AASIST-L training | Frozen configuration produces C0 scores |
| 6 | AASIST training | Frozen configuration produces C0 scores |
| 7 | Freeze models and thresholds; score all conditions | **G2:** predictions exist for every valid source-condition pair |
| 8 | Attribution analysis, cutoff curve, and sweep demonstration built from saved predictions | Tables and curves reproduce directly from saved predictions; demonstration runs offline on a laptop |
| 9 | Tier 2 — AASIST-L with band-limited augmentation, scored on all conditions | **G3:** recovery figure and clean-audio cost reported |
| 10–11 | Tier 3 if gates permit — WavLM system, on-device export | **G4:** completes cleanly or is reported as not attempted |
| 12 | Error analysis, report, model card, limitations, demo rehearsal | Claims match preregistered scope and intervals |

## Minimum viable completion

- Decomposition set and sweep set generated and integrity-checked
- At least two detectors scored across all conditions
- Attribution result with paired bootstrap intervals
- Cutoff curve with at least one collapse threshold and its interval
- The sweep demonstration, replayable offline from saved predictions
- Fixed-threshold error rates
- An explicit limitations section stating what was not attempted

Real calls, messaging applications, packet loss, replay, and unseen generators are outside the schedule at every tier.

## Demonstration

The demonstration is the cutoff sweep itself, and it is committed work in Tier 1.

One Thai utterance is replayed at successive cutoffs — the 16 kHz reference, then 6 kHz, 4 kHz, 3.4 kHz, 2.5 kHz, 1.5 kHz and 800 Hz — while the detector score for that utterance is plotted against the measured cutoff curve, with the 3.4 kHz telephone band edge and the collapse thresholds marked on the frequency axis. The audio makes the degradation audible and the curve makes the consequence visible at the same moment.

Requirements are deliberately small: the transformed sweep audio, the saved source-level predictions, and a plotting script. No GPU, no live inference, no export toolchain, and no optional tier. It survives every branch of the compute fallback because it replays results that already exist.

If Tier 3 also completes, the deployment arm extends the demonstration rather than replacing it: AASIST-L running on the target device, scoring band-limited and G.711-coded Thai audio in real time alongside the same curve.

Presentation rules:

- show the score and the frozen operating threshold, never a bare "real/fake" verdict;
- state the measured false-positive and false-negative rates at that threshold;
- use only project-owned or dataset-licensed audio in public demonstrations;
- never demonstrate on a bystander's voice without consent.

## Reproducibility checklist

- [ ] Dataset permissions, versions, and exclusions recorded
- [ ] Required ethics/SRC/IRB decision recorded
- [ ] Priority reads [4], [12], [19] completed and novelty position updated
- [ ] Storage location for restricted audio approved by licence and ethics decision
- [ ] Split and transformation manifest hashes saved
- [ ] G.711 implementation, resampler, and low-pass filter design pinned by version
- [ ] Resampler null check passed and recorded
- [ ] Sweep confirmed to run at a fixed 16 kHz sample rate throughout
- [ ] Detector revisions, configs, and random seeds saved
- [ ] Augmentation applied to training data only, never to the test manifest
- [ ] Development thresholds frozen before channel scoring
- [ ] Collapse-threshold definitions frozen before scoring
- [ ] Source-level predictions retained
- [ ] Bootstrap code moves all conditions of one source together
- [ ] Tables and curves regenerate from predictions without manual editing

## External resources

- ThaiSC LANTA overview: https://thaisc.io/thaisc-resorces/lanta
- ThaiSC services and access: https://thaisc.io/services
- YSC requirements: https://www.nstda.or.th/ysc/requirements/
- ITU-T G.711: https://www.itu.int/rec/T-REC-G.711/
