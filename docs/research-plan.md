# Research Plan: BandGap-TH

## 1. Problem statement

Thai voice-cloning scams arrive by telephone. The telephone path is narrowband: G.711 restricts the signal to roughly 300–3400 Hz [3]. Most vocoder and text-to-speech artifacts concentrate above that range, so a detector trained on clean wideband audio may be relying on evidence the telephone network deletes before the call arrives.

Published work establishes that codecs degrade detection — an average 5.30% EER degradation from clean to codec-compressed audio at zero packet loss [2], and real telephony transmission conditions in ASVspoof 2021 LA [10]. Thai telephony evaluation exists too: the CSS dataset paper reports baselines "investigated in telephony scenarios" [4], with the associated thesis extending to channel effects [12].

**What none of them separate is why.** A telephone codec does two independent things at once:

1. **Bandwidth reduction.** Low-pass filtering removes everything above roughly 3.4 kHz. The information is *deleted*.
2. **Companding.** Each sample is requantized from 16-bit linear to 8-bit logarithmic (A-law or μ-law). The information survives but acquires quantization noise.

Every published study applies the codec whole and reports a single degradation number, which is the sum of the two. The distinction is not academic — it determines what can be done about the problem:

| If damage is mostly… | Meaning | Remedy |
| --- | --- | --- |
| **Bandwidth** | The evidence no longer exists in the signal | No training recovers deleted information. Requires cues inside 300–3400 Hz, or wider-band infrastructure |
| **Companding** | The evidence survives, buried in quantization noise | Recoverable through augmentation or noise-robust representations |

### Research questions

1. **Attribution.** Of the EER increase caused by simulated G.711 coding on Thai audio, how much is caused by bandwidth reduction and how much by A-law/μ-law companding?
2. **Dose–response.** How much bandwidth can be removed before Thai deepfake detection collapses, and at what cutoff?
3. **Mitigation.** Does training on band-limited audio recover the loss, and by how much?

Together these form diagnosis, quantification, and attempted remedy.

### Hypotheses

- **H1 — attribution:** bandwidth reduction contributes more to G.711 degradation than companding does.
- **H2 — dose–response:** EER increases monotonically as the low-pass cutoff decreases, with an identifiable collapse threshold.
- **H3 — mitigation:** training with band-limited augmentation reduces `ΔEER` at telephone conditions relative to a clean-trained baseline of identical architecture.

All three are falsifiable and informative in either direction. If H1 is rejected and companding dominates, the problem is far more tractable than expected — a more useful and more surprising result than the predicted one. If H3 is rejected, that is direct evidence that the loss is informational rather than a training artifact, which strengthens the bandwidth interpretation.

## 2. Honest novelty position

Recorded so that no claim overstates its contribution. Revise, do not delete, if week-1 reading changes the picture.

**Already published, not claimed here:**

- **Thai telephony evaluation** [4, 12]. The channel conditions in this project replicate published work, and are described as replication throughout.
- **Real telephony codec conditions** for English [10].
- **Codec robustness benchmarking** — ADD-C covers six wideband codecs and excludes narrowband G.711 [2].
- **Threshold-transfer failure** [19]. Reported here as a secondary check, citing that paper.
- **Efficient anti-spoofing architectures** [7, 11].

**Claimed here:**

1. **Bandwidth-versus-companding attribution.** Not done in any language. The C1 control is original.
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

C1 is the control that makes attribution possible. It removes the high band while retaining full 16-bit precision, so subtracting it from C2 and C3 isolates the companding contribution. Without C1, only the sum is observable.

### Sweep set — dose–response

| ID | Low-pass cutoff |
| --- | --- |
| S8000 | 8000 Hz |
| S6000 | 6000 Hz |
| S4000 | 4000 Hz |
| S3400 | 3400 Hz (telephone band edge) |
| S2500 | 2500 Hz |
| S1500 | 1500 Hz |
| S800 | 800 Hz |

**The sweep never changes sample rate.** Every sweep condition stays at 16 kHz throughout and applies only a low-pass filter, so the cutoff frequency is the sole variable and the resampler is removed from the experiment entirely. This is a deliberate design choice: the decomposition set must round-trip through 8 kHz because G.711 requires it, but the sweep does not, and keeping the rate fixed eliminates a confound rather than controlling for it.

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

estimated with a paired cluster bootstrap whose resampling unit is the source recording. All conditions of one source move together in every replicate, which is what makes the contrast well powered — the sample size is the number of recordings, not the number of conditions. H1 is supported if the interval excludes zero from above.

The decomposition is reported per detector and as a fraction of the total effect, so that "bandwidth accounts for N% of G.711 degradation" is stated with an interval.

Additivity is not assumed. `Companding part` is strictly the additional effect of companding *given* bandwidth already reduced — the relevant quantity for deployment, since real telephony always applies both.

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

The project succeeds scientifically whether hypotheses are supported or rejected, provided the protocol completes without leakage and uncertainty is reported.

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

- Attribution gap positive, interval excluding zero → bandwidth dominates; the evidence is deleted and mitigation is bounded by information loss.
- Attribution gap negative → companding dominates; the problem is substantially recoverable, and this is the more actionable outcome.
- A sharp collapse threshold near 3.4 kHz → narrowband telephony is the operative limit for deployed detection.
- A gradual curve with no knee → no single cutoff explains failure, and the framing must be revised.
- Recovery high with low clean-audio cost → augmentation is a practical fix.
- Recovery near zero → direct evidence that the loss is informational, reinforcing the bandwidth interpretation.
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
- Leakage audit, model card, ethics statement, and limitations

See [`feasibility.md`](feasibility.md) for the tiered schedule and [`compute-request.md`](compute-request.md) for the resource estimate.
