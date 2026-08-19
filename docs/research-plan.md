# Research Plan: BandGap-TH

## 1. Problem statement

Thai voice-cloning scams often reach victims by telephone. A conventional G.711 telephone path carries a narrow voice-frequency band of roughly 300–3400 Hz [3]. Some artifacts used to distinguish synthetic speech may lie outside that range, so a detector trained on clean wideband audio may rely on information that is missing from telephone audio.

Published work shows that codecs can degrade detection. One study reports an average 5.30% EER degradation from clean to codec-compressed audio at zero packet loss [2], while ASVspoof 2021 LA includes real telephony transmission conditions [10]. Thai telephony evaluation also exists: the CSS dataset paper reports baselines "investigated in telephony scenarios" [4], and the associated thesis examines channel effects [12].

However, these studies do not clearly separate two effects that occur together in a G.711 path:

1. **Bandwidth reduction.** Low-pass filtering removes frequencies above roughly 3.4 kHz, so information in that band is no longer available to the detector.
2. **Companding.** Each sample is requantized from 16-bit linear to 8-bit logarithmic A-law or μ-law, changing the signal through quantization.

Studies usually apply the complete codec and report one overall degradation value. Separating the two contributions helps clarify what kind of mitigation may be useful:

| If damage is mostly… | Meaning | Remedy |
| --- | --- | --- |
| **Bandwidth** | Information in the removed band is unavailable | Use cues within 300–3400 Hz or preserve a wider band |
| **Companding** | The signal remains but is altered by quantization | Channel-aware augmentation or more robust representations may help |

### Research questions

1. **Attribution.** Of the EER increase under simulated G.711 coding on Thai audio, how much is associated with bandwidth reduction and how much with A-law/μ-law companding?
2. **Dose-response.** How does detection performance change as the low-pass cutoff is reduced, and at what cutoff does it cross the preregistered failure thresholds?
3. **Mitigation.** Does training with band-limited audio recover some of the loss, and by how much?

### Hypotheses

- **H1 — attribution:** bandwidth reduction contributes more to G.711 degradation than companding does.
- **H2 — dose–response:** EER increases monotonically as the low-pass cutoff decreases, with an identifiable collapse threshold.
- **H3 — mitigation:** training with band-limited augmentation reduces `ΔEER` at telephone conditions relative to a clean-trained baseline of identical architecture.

All three hypotheses can be supported or rejected. If H1 is rejected and companding contributes more than bandwidth reduction, channel-aware training may have more room to help than expected. If H3 is rejected, the result would be consistent with information loss from bandwidth reduction, although other explanations would still need to be examined.

## 2. Novelty position

This section separates earlier work from the claims made by BandGap-TH. It must be revised if the week-1 literature review changes the novelty assessment.

**Already published, not claimed here:**

- **Thai telephony evaluation** [4, 12]. The channel conditions in this project replicate published work, and are described as replication throughout.
- **Real telephony codec conditions** for English [10].
- **Codec robustness benchmarking** — ADD-C covers six wideband codecs and excludes narrowband G.711 [2].
- **Threshold-transfer failure** [19]. Reported here as a secondary check, citing that paper.
- **Efficient anti-spoofing architectures** [7, 11].

**Claimed here:**

1. **Bandwidth-versus-companding attribution.** We found no published study that reports this attribution directly. The C1 control is the proposed contribution.
2. **A cutoff dose–response curve with a collapse threshold** for Thai audio-deepfake detection.
3. **A measured recovery figure** for band-limited augmentation against that curve.

## 3. Experimental conditions

### Decomposition set — attribution

All four conditions are paired: every test source produces all four.

| ID | Condition | Contains |
| --- | --- | --- |
| C0 | Original 16 kHz linear PCM | reference |
| C1 | 8 kHz **linear PCM** round trip, resampled back to 16 kHz | bandwidth only |
| C2 | G.711 μ-law encode/decode at 8 kHz, resampled back | bandwidth + companding |
| C3 | G.711 A-law encode/decode at 8 kHz, resampled back | bandwidth + companding |

C1 is the bandwidth-only control. It removes the high-frequency band while retaining 16-bit linear precision. Comparing C1 with C2 and C3 estimates the additional change associated with companding after bandwidth has already been reduced.

### Sweep set — dose–response

| ID | Low-pass cutoff |
| --- | --- |
| S8000 | No additional low-pass filter (16 kHz reference; nominal 8 kHz Nyquist limit) |
| S6000 | 6000 Hz |
| S4000 | 4000 Hz |
| S3400 | 3400 Hz (telephone band edge) |
| S2500 | 2500 Hz |
| S1500 | 1500 Hz |
| S800 | 800 Hz |

Every sweep condition remains at a 16 kHz sample rate and applies only a low-pass filter. The cutoff frequency is therefore the only planned variable in the sweep, and resampling is not introduced. The decomposition set still requires an 8 kHz round trip to represent G.711, but the sweep does not.

One filter design, one transition-band specification, and one implementation are used for every sweep point, pinned by version.

### Transformation integrity

- Each transformed file maps to exactly one source ID.
- Duration drift stays within a preregistered tolerance.
- All conditions contain the same source content and labels as C0.
- Hashes of transformation configuration and output manifests are saved.
- Genuine and spoof counts are identical across paired conditions.
- Every condition is applied symmetrically to genuine and spoof audio.
- A random sample is decoded and inspected before scoring.

### Resampler null check

The decomposition set round-trips through 8 kHz, so the resampler could itself introduce a class-separable artifact and be misattributed to bandwidth. Before any scoring, a null check confirms that a 16 kHz → 8 kHz → 16 kHz round trip applied to already-narrowband audio does not create a detectable cue. The sweep set is immune by construction.

## 4. Dataset design

Potential Thai sources are Chula Spoofed Speech (CSS) and SEA-Spoof, subject to access approval and license [4–6].

**Week-1 priority reads:** [4], [12] and [19]. These determine how much of this project is novel. Update section 2 against what they actually report — which codecs, which conditions, which detectors.

### Split controls

- Speaker-disjoint train, development, and test partitions
- Script/text-disjoint partitions where metadata permits
- Reference-audio-disjoint partitions for cloned speech
- Generator-checkpoint-disjoint partitions where metadata permits
- No transformed version of a test source in training or development
- One immutable test manifest used for every detector and condition

### Fallback

If restricted Thai data is unavailable, all three questions remain answerable on lawfully accessible ASVspoof data, since none is language-specific. Such a result is labelled non-Thai and the report states that the Thai claim was not tested.

## 5. Detector panel

The panel exists to show that findings are not an artifact of one architecture. It is deliberately small.

| System | Approx. scale | Role | Tier |
| --- | --- | --- | --- |
| LFCC-GMM | non-neural | classical floor, cheap sanity check | 1 |
| AASIST-L | ~85K parameters | on-device candidate; the mitigation subject | 1 |
| AASIST | ~297K parameters | small neural reference | 1 |
| WavLM + AASIST back-end | ~94M parameters | large self-supervised reference | 2, optional |

No new architecture is proposed. The large system is included only to check whether self-supervised pretraining changes the shape of the curve; if compute is short it is dropped and that is reported as not tested.

Hyperparameters are selected on clean training/development data and frozen. No channel or sweep result may influence architecture, training schedule, threshold, or preprocessing. Three seeds per trainable system where compute permits.

## 6. Evaluation protocol

### Attribution analysis

```text
EER_G711        = mean(EER_C2, EER_C3)
Total effect    = EER_G711 − EER_C0
Bandwidth part  = EER_C1   − EER_C0
Companding part = EER_G711 − EER_C1
```

H1 is tested as a paired contrast:

```text
Attribution gap = Bandwidth part − Companding part
```

The attribution gap is estimated with a paired cluster bootstrap whose resampling unit is the source recording. All conditions created from one source move together in every replicate. H1 is supported if the confidence interval is entirely above zero.

The decomposition is reported for each detector and as a fraction of the total effect, with an interval around the estimated percentage attributed to bandwidth.

The analysis does not assume that bandwidth reduction and companding are independent or additive. `Companding part` is the additional change observed after bandwidth has already been reduced.

### Dose–response analysis

`EER(cutoff)` is reported for every sweep point with paired bootstrap intervals, per detector, as a curve.

Two collapse thresholds are preregistered and estimated by interpolation with bootstrap intervals on the threshold itself:

- **Relative:** the cutoff at which EER first reaches twice the clean EER.
- **Absolute:** the cutoff at which EER first reaches 10%.

Monotonicity is tested rather than assumed; any non-monotonicity is reported rather than smoothed.

### Mitigation analysis

One architecture — AASIST-L, the deployment candidate — is retrained with band-limited augmentation: low-pass cutoff sampled from the sweep range, plus G.711 companding, applied to training data only. Everything else is held identical to the baseline: same splits, same schedule, same seeds, same stopping rule.

```text
Recovery = (ΔEER_baseline − ΔEER_augmented) / ΔEER_baseline
```

reported at the G.711 condition and across the sweep curve, with paired bootstrap intervals. H3 is supported if recovery is positive with an interval excluding zero.

The relative recovery ratio is reported only when `ΔEER_baseline` is positive and large enough to avoid an unstable denominator. If the baseline degradation is zero or negative, the study reports the absolute difference in `ΔEER` instead and marks the ratio as undefined.

The augmented model is also evaluated on **clean** audio, to report the cost of augmentation rather than only its benefit.

### Threshold-transfer check — secondary, not claimed

Following [19], each detector's operating threshold is selected once on the clean development set and transferred unchanged to all conditions, reporting false-positive rate, false-negative rate, balanced accuracy, and change relative to C0. The argument and its strongest evidence belong to [19].

### Secondary metrics

AUROC, AUPRC, EER separately for C2 and C3, per-generator and per-speaker breakdowns where sample size and privacy permit, and — for the deployment candidate — parameter count, MACs per second of audio, real-time factor and peak memory on the declared target device.

## 7. Leakage and confounding controls

- Apply every condition symmetrically to genuine and spoof samples.
- Use identical containers, input rates, duration handling, and file naming across classes.
- Pin one resampler configuration and one filter design; vary only the cutoff in the sweep.
- Keep the sweep at a fixed 16 kHz sample rate so resampling is absent, not merely controlled.
- Apply augmentation to training data only, never to the test manifest.
- Do not allow condition labels to correlate with class labels.
- Freeze all thresholds before opening transformed test results.
- Store source IDs separately from human-readable identities.
- Audit duplicate or near-duplicate audio before splitting.
- Report exclusions and complete sample counts in every condition.

## 8. Decision rules

The results remain useful whether the hypotheses are supported or rejected, provided the protocol is completed without leakage and uncertainty is reported.

Preregister before final scoring:

1. the immutable test manifest;
2. detector configurations;
3. the seed policy;
4. all transformation commands, the pinned resampler, and the pinned filter design;
5. the sweep cutoffs;
6. both collapse-threshold definitions;
7. the augmentation distribution;
8. bootstrap units and replicate count; and
9. handling of failed or corrupted samples.

Interpretation:

- A positive attribution gap with an interval entirely above zero supports a larger bandwidth contribution.
- A negative attribution gap supports a larger companding contribution and motivates further channel-aware training experiments.
- A sharp failure threshold near 3.4 kHz would support the claim that narrow telephone bandwidth limits these detectors under the tested conditions.
- A gradual curve with no clear knee would show that no single cutoff explains the performance change.
- High recovery with little loss on clean audio would support augmentation as a practical mitigation for the tested model.
- Recovery near zero would be consistent with information loss, but would not prove that bandwidth is the only explanation.
- Wide intervals produce an inconclusive result, not evidence of no effect.

## 9. Limitations

- Simulated G.711 is not a real telephone network. Real calls add packet loss, jitter, transcoding, handset characteristics, room acoustics, and gain control, none of which are studied here.
- The channel conditions replicate published Thai telephony work [4, 12].
- Low-pass filtering is a model of bandwidth limitation, not an exact reproduction of any specific network's frequency response.
- Attribution is conditional, not a decomposition into independent additive causes.
- Collapse thresholds are specific to the detectors and dataset used.
- Dataset access and Thai speaker diversity may constrain external validity.
- EER does not represent real scam prevalence or asymmetric deployment costs.
- Results cannot establish that any detector is safe for forensic or financial decisions.

## 10. Deliverables

- Versioned source and transformed manifests
- Reproducible transformation scripts for decomposition and sweep sets
- Frozen detector configurations, augmentation recipe, and model revisions
- Source-level prediction files for every detector and condition
- Attribution, dose–response, mitigation, and bootstrap analysis scripts
- The cutoff curve with collapse thresholds and intervals
- The sweep demonstration: paired audio and score replay against the curve, running offline from saved predictions
- Leakage audit, model card, ethics statement, and limitations

See [`feasibility.md`](feasibility.md) for the tiered schedule and [`compute-request.md`](compute-request.md) for the resource estimate.
