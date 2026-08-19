# Research Plan: ScaleGap-TH

## 1. Problem statement

Audio-deepfake detectors are increasingly deployed at the edge — on phones, gateways, and single-board computers — where compute is scarce. The standard response is to use a smaller model. That advice is validated almost entirely on clean, wideband benchmark audio.

Thai voice-cloning scams do not arrive as clean wideband audio. They arrive over a telephone path, where G.711 low-pass filters the signal to roughly 3.4 kHz [3]. Most vocoder and TTS artifacts concentrate above that cutoff. This creates a specific, testable concern: a high-capacity detector may retain redundant discriminative cues in the surviving band, while a small detector may have allocated its limited capacity to the band the channel removes. If so, the accuracy cost of shrinking a detector is larger on telephone audio than on clean audio, and edge deployment is harder than clean-condition efficiency numbers suggest.

### What prior work does and does not establish

Müller et al. reported RawNet2 rising from 3.154% EER on ASVspoof 2019 LA to 37.819% on an In-the-Wild dataset, with roughly 200–1,000% EER deterioration across configurations [1]. That measures domain shift, not capacity.

Shi et al. benchmarked three detector families across six communication codecs and five packet-loss rates, reporting an average 5.30% EER degradation from clean audio to codec-compressed audio at zero packet loss, and proposed a data-augmentation remedy [2]. Their codec set — AMR-WB, EVS, IVAS, Opus, Speex-WB and SILK — is entirely wideband. Narrowband G.711 is not evaluated.

ASVspoof 2021 LA does include real telephony transmission: condition C3 traverses a PSTN path with a μ-law codec at 8 kHz, and further conditions traverse a PBX using a-law and G.722 among others [10]. This is stronger evidence than simulation, but it is English, it reports no capacity axis, and it provides no bandwidth-only control that would separate low-pass filtering from companding.

On the efficiency side, AASIST-L attains competitive accuracy at roughly 85K parameters [7], and subsequent work sparsifies the AASIST backend specifically for deployment [11]. Both characterise the accuracy–compute trade-off under clean conditions.

The interaction is unmeasured. No published work reports EER degradation as a function of detector capacity under narrowband telephone coding, and none decomposes that degradation into bandwidth and companding components.

### Research question

> Does G.711 telephone coding degrade small-capacity Thai audio-deepfake detectors more than large-capacity ones?

### Hypotheses

- **H1 — frontier steepening (primary):** the increase in EER caused by G.711 coding is larger for small-capacity detectors than for large-capacity detectors.
- **H2 — absolute degradation:** G.711 coding increases EER relative to paired clean audio for most detector families.
- **H3 — bandwidth attribution:** some degradation is explained by the 8 kHz bandwidth reduction alone, while additional degradation may be attributable to A-law/μ-law companding.
- **H4 — deployment viability:** at least one detector under 100K parameters retains usable G.711 performance while meeting the preregistered real-time factor and peak-memory budget on the declared target device.

The study is descriptive and falsifiable in both directions. H1 is unsupported if the difference-in-differences interval includes or falls below zero — which would be a useful result, indicating that cheap detectors are safe for this channel. H2 is unsupported if paired intervals show no positive degradation. H3 is unsupported if G.711 conditions do not differ meaningfully from the 8 kHz linear-PCM control. H4 is unsupported if no small detector meets both budgets.

## 2. Contribution and scope

The contribution is a controlled, paired measurement of how the accuracy–compute frontier for Thai audio-deepfake detection changes under a narrowband telephone channel, plus an on-device measurement establishing whether the small end of that frontier is reachable in practice. It combines:

1. identical source recordings across all channel conditions;
2. symmetric transformation of genuine and spoof classes;
3. an 8 kHz bandwidth-only control;
4. a detector panel spanning roughly three orders of magnitude in capacity;
5. fixed development thresholds transferred to channel conditions;
6. paired difference-in-differences estimates with cluster-bootstrap uncertainty; and
7. measured compute cost on declared target hardware.

### Explicit exclusions

The committed study does not propose a new architecture, does not pursue state of the art on any benchmark, and does not investigate unseen generators, messaging applications, packet loss, replay, background noise, real telephone calls as a main condition, partial deepfakes, abstention, or Thai-tone auxiliaries. These may be discussed as future work but cannot enter the main experiment after preregistration.

Designing a novel nano architecture is deliberately out of scope. AASIST-L already occupies the sub-100K size class [7] and a sparsified successor exists [11]; attempting to outperform them is a poor use of the schedule and would replace a falsifiable question with an engineering race.

## 3. Experimental conditions

Every test source produces four deterministic, paired conditions:

| ID | Condition | Purpose |
| --- | --- | --- |
| C0 | Original 16 kHz linear PCM | Clean benchmark reference |
| C1 | 8 kHz linear PCM round trip, then resampled to model rate | Isolate telephone-bandwidth reduction |
| C2 | G.711 μ-law encode/decode at 8 kHz, then resampled | Measure μ-law path |
| C3 | G.711 A-law encode/decode at 8 kHz, then resampled | Measure A-law path |

G.711 is formally defined by ITU-T Recommendation G.711 [3]. The exact encoder, decoder, resampler, software version, and command line will be frozen before final evaluation.

**Resampler control.** Because C1–C3 are resampled back up to the model input rate, the resampling filter is itself a potential artifact and a potential confound for the bandwidth effect. One resampler, one filter design, and one set of parameters will be used for every condition and both classes, pinned by version and recorded in the transformation manifest. A null check will confirm that a C0→8 kHz→16 kHz round trip applied to already-narrowband audio does not itself create a class-separable cue.

Peak normalization, denoising, silence trimming, and loudness normalization will not be applied differently across conditions.

### Transformation integrity checks

- Each transformed file must map to exactly one source ID.
- Duration drift must remain within a preregistered tolerance.
- C1–C3 must contain the same source content and labels as C0.
- Hashes of transformation configuration and output manifests must be saved.
- Genuine and spoof counts must be identical across paired conditions.
- A random sample will be decoded and inspected for corruption before scoring.

## 4. Dataset design

Potential Thai sources include Chula Spoofed Speech (CSS) and SEA-Spoof, subject to current access approval and licenses [4–6]. The final dataset choice must be frozen before model comparison.

**Prior-work check.** The thesis underlying the CSS release is titled to include channel effects [12]. It must be read in full before the protocol is frozen. If it already reports narrowband telephone conditions on CSS, this project's channel protocol is a replication and the capacity axis becomes the sole contribution; the proposal will say so explicitly rather than overstate novelty.

### Split controls

- Speaker-disjoint train, development, and test partitions
- Script/text-disjoint partitions where metadata permits
- Reference-audio-disjoint partitions for cloned speech
- Generator-checkpoint-disjoint partitions where metadata permits
- No transformed version of a test source in training or development
- One immutable test manifest used for every detector and condition

Generator identity is stratified where possible but is not a generalization target.

### Dataset fallback

If restricted Thai data is unavailable, the capacity-versus-channel question remains answerable on lawfully accessible ASVspoof data. This is a genuine fallback rather than a dead end: the frontier hypothesis is not language-specific, so a completed non-Thai study still answers H1–H4. Such a result must be labelled non-Thai, and the report will state plainly that the Thai application claim was not tested.

## 5. Detector panel

Capacity is the manipulated variable. The panel uses published systems only, spanning roughly three orders of magnitude:

| System | Approx. scale | Role | Ref |
| --- | --- | --- | --- |
| LFCC-GMM | non-neural | classical floor | — |
| AASIST-L | ~85K parameters | on-device candidate | [7] |
| AASIST | ~297K parameters | small neural reference | [7] |
| Frozen WavLM + linear head | ~94M frozen | large fixed representation | [8] |
| WavLM + single LoRA adapter | ~94M + adapters | large adapted | [8, 9] |

Parameter counts above are indicative. Actual parameter counts, MACs per second of audio, and measured latency are reported from the frozen configurations.

Every detector receives identical train/development/test source partitions. Hyperparameters are selected using only the clean training/development data and then frozen. Channel-condition test scores cannot be used to tune architectures, training schedules, thresholds, or preprocessing.

Three independent training seeds will be used for trainable neural systems if compute permits. Seed-level results quantify optimization variability and are reported as such.

### Confounding between capacity and architecture

Capacity cannot be varied in isolation across published systems: AASIST-L and WavLM differ in family as well as size. The design addresses this rather than ignoring it, by preregistering two contrasts:

- **Range contrast (primary):** AASIST-L versus WavLM+LoRA. Largest capacity span, confounded with family.
- **Within-family contrast (secondary):** AASIST-L versus AASIST. Family held constant, narrower span.

Agreement between the two strengthens a capacity interpretation. Disagreement will be reported as evidence that family, not size, drives the effect.

## 6. Evaluation protocol

### Primary condition

For detector `m`:

```text
EER_G711(m) = mean(EER_C2(m), EER_C3(m))
ΔEER_G711(m) = EER_G711(m) − EER_C0(m)
```

The macro-average prevents a decision based on whichever G.711 law happens to be easier.

### Primary analysis — difference in differences

```text
DiD(small, large) = ΔEER_G711(small) − ΔEER_G711(large)
```

Both detectors are evaluated on the same paired recordings, so the contrast is estimated with a paired cluster bootstrap whose resampling unit is the original source recording. All channel versions of one source, and all detectors' scores for that source, move together in every replicate. H1 is supported if the 95% interval for `DiD` excludes zero from above.

Where repeated speech from the same speaker creates dependence, a speaker-cluster sensitivity analysis will also be reported.

### Frontier reporting

`ΔEER_G711` is reported for all five systems against measured capacity, as a descriptive Pareto curve with per-system intervals. With five systems, a fitted slope over capacity has low inferential power; it will be reported as an effect estimate with uncertainty, never reduced to a p-value.

### Bandwidth attribution

```text
Bandwidth effect          = EER_C1 − EER_C0
Additional μ-law effect   = EER_C2 − EER_C1
Additional A-law effect   = EER_C3 − EER_C1
```

This avoids attributing all narrowband degradation to companding. The decomposition is reported per system, so that H1 can be examined separately for the bandwidth and companding components.

### Threshold-transfer analysis

EER permits a new threshold for each condition, which can hide deployment failure. Each detector's operating threshold will also be selected once on the clean development set and transferred unchanged to C0–C3. Report:

- false-positive rate for genuine audio;
- false-negative rate for spoof audio;
- balanced accuracy; and
- change relative to C0.

### Compute measurement

For every detector, measured and reported:

- parameter count and model file size
- multiply–accumulate operations per second of audio
- real-time factor on the declared target device
- peak resident memory under sustained inference
- the same figures on the development workstation, for comparability

Measurement conditions — device, thermal state, batch size, input length, number of repetitions, and how the median and spread are computed — are preregistered.

### Secondary metrics

- AUROC and AUPRC
- EER separately for C2 and C3
- Per-generator and per-speaker breakdowns where sample size and privacy permit

## 7. Leakage and confounding controls

- Apply C1–C3 symmetrically to genuine and spoof samples.
- Use identical containers, input rates, duration handling, and file naming across classes.
- Use one pinned resampler configuration everywhere.
- Do not encode only one class or generator family.
- Do not allow channel labels to correlate with class labels.
- Freeze all thresholds before opening transformed test results.
- Store source IDs separately from human-readable identities.
- Audit duplicate or near-duplicate audio before splitting.
- Report exclusions and the complete sample count in every condition.

## 8. Decision rules

The project succeeds scientifically whether the hypotheses are supported or rejected, provided the protocol is completed without leakage and uncertainty is reported.

Before final scoring, preregister:

1. the immutable test manifest;
2. the five detector configurations;
3. the seed policy;
4. C0–C3 transformation commands and the pinned resampler;
5. the primary G.711 macro-average and the two DiD contrasts;
6. bootstrap units and replicate count;
7. the target device, the real-time-factor budget, and the peak-memory budget for H4; and
8. handling of failed or corrupted samples.

Interpretation:

- A positive `DiD` whose interval excludes zero supports frontier steepening.
- A `DiD` interval containing zero does not support steepening and will be reported as such, not as a null to be explained away.
- A negative `DiD` supports the opposite conclusion: small detectors are comparatively robust to this channel.
- Agreement between the range and within-family contrasts supports a capacity interpretation; disagreement supports an architecture-family interpretation.
- Similar C1, C2, and C3 results indicate bandwidth — not G.711 companding — is the dominant measured factor.
- Wide intervals produce an inconclusive result, not evidence of no effect.

## 9. Limitations

- Simulated G.711 is not a complete telephone network.
- Capacity is entangled with architecture family; the within-family contrast narrows but does not eliminate this.
- Five systems provide only a preliminary frontier shape.
- Dataset access and Thai speaker diversity may constrain external validity.
- EER does not represent real scam prevalence or asymmetric deployment costs.
- On-device timings are specific to one device, one runtime, and one thermal environment.
- Current generators may not represent future synthesis systems.
- Results cannot establish that any detector is safe for forensic or financial decisions.

## 10. Deliverables

- Versioned source and transformed manifests
- Reproducible C0–C3 transformation script and environment
- Frozen detector configurations and model revisions
- Source-level prediction files
- EER, threshold-transfer, DiD and bootstrap analysis scripts
- Frontier plots of `ΔEER_G711` against measured capacity
- On-device benchmark script and measured latency/memory tables
- Leakage audit, model card, ethics statement, and limitations

See [`feasibility.md`](feasibility.md) for the schedule, compute plan, and hardware plan.
