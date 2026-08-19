# Research Plan: ScaleGap-TH

## 1. Problem statement

Audio-deepfake detectors are increasingly deployed at the edge — on phones, gateways, and single-board computers — where compute is scarce. The standard response is to use a smaller model, and on clean benchmark data that looks free: AASIST reports 0.83% EER on ASVspoof 2019 LA at 297K parameters, while AASIST-L reports 0.99% at 85K [7]. A 3.5× reduction costs 0.16 percentage points.

Away from the benchmark the picture inverts. Detectors that score below 1% in-domain reach 30–60% EER on in-the-wild corpora, and RawNet2 exceeds 40% EER on most out-of-distribution sets, while large self-supervised systems remain in the 7–9% range [20, 21]. On that evidence, size appears to buy robustness that the benchmark cannot see.

**But size is confounded with pretraining.** The robust systems are large *because* they are built on self-supervised encoders pretrained on tens of thousands of hours of unlabeled speech. Their robustness may come from the pretraining corpus rather than from parameter count, and the effect is not monotone in size — a 317M-parameter Wav2Vec2-AASIST does not automatically outperform smaller systems on cross-dataset leaderboards [21].

No published work separates these two factors. That matters directly for deployment: if robustness comes from parameter count, edge deployment is bounded by the device. If it comes from pretraining, a small *distilled* detector could inherit it, and the edge constraint largely dissolves.

Thai voice-cloning scams arrive over the telephone, where G.711 low-pass filters the signal to roughly 3.4 kHz [3] — removing the band where most vocoder and TTS artifacts concentrate. This project asks the size-versus-pretraining question under exactly that channel.

### Research question

> Does Thai telephone-channel robustness in audio-deepfake detection come from model size or from self-supervised pretraining, and can a phone-sized detector inherit it?

## 2. Honest novelty position

This section records what is already published, so that no claim in this project overstates its contribution. It must be revised, not deleted, if week-1 reading changes the picture.

**Already published, and not claimed here:**

- **Thai telephony evaluation.** The CSS dataset paper states that its baselines "were investigated in telephony scenarios" and that the dataset supports Thai anti-spoofing "in real-world telephony situations" [4]. The associated thesis extends to channel effects [12]. **The channel protocol in this project is therefore a replication, not a new finding**, and is described as such throughout.
- **Real telephony codec conditions.** ASVspoof 2021 LA includes real PSTN and PBX transmission using μ-law and a-law [10], for English.
- **Codec robustness benchmarking.** ADD-C covers six wideband codecs — AMR-WB, EVS, IVAS, Opus, Speex-WB and SILK — and excludes narrowband G.711 [2].
- **Threshold-transfer failure.** That EER at an oracle threshold hides deployment failure is established: a 0.21%-EER detector operated at its source threshold rejects 78.7% of genuine in-the-wild speech while its target EER still reads 11.2% [19]. **This project does not claim the threshold-transfer argument.** It reports transferred-threshold error rates as a secondary deployment check, citing [19], and adopts that paper's recommendation to report error at a transferred threshold alongside EER.
- **Cross-dataset and cross-language generalization gaps** [20, 21, 22].
- **Efficient anti-spoofing architectures.** AASIST-L at ~85K parameters [7], and a sparsified AASIST backend for deployment [11].

**What remains genuinely open, and is claimed here:**

1. **Size versus pretraining, disentangled.** No published work holds pretraining constant while varying size, or holds size constant while varying pretraining, for channel robustness. This is the primary contribution.
2. **Bandwidth versus companding decomposition.** No published work separates the effect of 3.4 kHz low-pass filtering from A-law/μ-law companding, in any language. The C1 control is original.
3. **On-device measurement under channel conditions for Thai.** Not previously reported.

## 3. Design: three cells, two clean contrasts

Capacity and pretraining are varied independently across the detector panel:

|  | From-scratch | SSL-pretrained |
| --- | --- | --- |
| **Small** (<1M params) | AASIST-L (~85K) | **Distilled WavLM student** (~1M) |
| **Large** (>90M params) | *(no standard system exists)* | WavLM + AASIST back-end (~94M) |

The lower-left cell is empty because no standard large from-scratch anti-spoofing system exists; this is a stated limitation, not an oversight. The three occupied cells still support two clean, independently interpretable contrasts:

- **Pretraining effect, size held approximately constant:** AASIST-L versus the distilled student. Both are small and deployable; they differ in whether the representation was self-supervised-pretrained.
- **Size effect, pretraining held constant:** distilled student versus WavLM + AASIST. Both inherit the same pretrained representation; they differ by roughly two orders of magnitude in size.

The diagonal — AASIST-L versus WavLM + AASIST — is the confounded contrast that the existing literature reports. Here it is interpretable as approximately the sum of the two clean effects, which provides an internal consistency check.

### Hypotheses

- **H1 — pretraining effect (primary):** at matched small size, the SSL-distilled detector degrades less under G.711 than the from-scratch detector.
- **H2 — size effect (primary):** at matched pretraining, the small distilled student degrades more under G.711 than the full-size SSL system.
- **H3 — absolute degradation:** G.711 coding increases EER relative to paired clean audio for most detectors.
- **H4 — bandwidth attribution:** some degradation is explained by 8 kHz bandwidth reduction alone, with any remainder attributable to A-law/μ-law companding.
- **H5 — deployment viability:** the distilled student meets the preregistered real-time factor and peak-memory budget on the declared target device while retaining usable G.711 performance.

The interesting outcomes are asymmetric and all informative. H1 supported and H2 unsupported means pretraining carries the robustness and small distilled detectors are viable at the edge — the most actionable result. H1 unsupported and H2 supported means capacity is irreducible and edge deployment is genuinely constrained. Both supported means the two factors contribute separately, and the frontier is steeper than either alone suggests.

## 4. Experimental conditions

Every test source produces four deterministic, paired conditions:

| ID | Condition | Purpose |
| --- | --- | --- |
| C0 | Original 16 kHz linear PCM | Clean benchmark reference |
| C1 | 8 kHz linear PCM round trip, then resampled to model rate | Isolate telephone-bandwidth reduction |
| C2 | G.711 μ-law encode/decode at 8 kHz, then resampled | Measure μ-law path |
| C3 | G.711 A-law encode/decode at 8 kHz, then resampled | Measure A-law path |

G.711 is defined by ITU-T Recommendation G.711 [3]. The exact encoder, decoder, resampler, software version, and command line are frozen before final evaluation.

**Resampler control.** Because C1–C3 are resampled back up to the model input rate, the resampling filter is itself a potential artifact and a confound for the bandwidth effect. One resampler, one filter design, and one parameter set is used for every condition and both classes, pinned by version and recorded in the transformation manifest. A null check confirms that a C0→8 kHz→16 kHz round trip applied to already-narrowband audio does not itself create a class-separable cue.

Peak normalization, denoising, silence trimming, and loudness normalization are not applied differently across conditions.

### Transformation integrity checks

- Each transformed file maps to exactly one source ID.
- Duration drift stays within a preregistered tolerance.
- C1–C3 contain the same source content and labels as C0.
- Hashes of transformation configuration and output manifests are saved.
- Genuine and spoof counts are identical across paired conditions.
- A random sample is decoded and inspected for corruption before scoring.

## 5. Dataset design

Potential Thai sources include Chula Spoofed Speech (CSS) and SEA-Spoof, subject to access approval and licenses [4–6].

**Week-1 prior-work requirement.** The CSS paper [4] and thesis [12] must be read in full before the protocol is frozen, and the novelty position in section 2 updated against what they actually report — including which codecs, which conditions, and which detectors. If they already report the exact G.711 conditions used here, this project cites them as the baseline and reports its own channel results as a replication supporting the capacity and pretraining contrasts.

### Split controls

- Speaker-disjoint train, development, and test partitions
- Script/text-disjoint partitions where metadata permits
- Reference-audio-disjoint partitions for cloned speech
- Generator-checkpoint-disjoint partitions where metadata permits
- No transformed version of a test source in training or development
- One immutable test manifest used for every detector and condition

### Dataset fallback

If restricted Thai data is unavailable, the size-versus-pretraining question remains answerable on lawfully accessible ASVspoof data, since it is not language-specific. Such a result is labelled non-Thai, and the report states plainly that the Thai application claim was not tested.

## 6. Detector panel

| System | Approx. scale | Cell | Ref |
| --- | --- | --- | --- |
| LFCC-GMM | non-neural | classical floor | — |
| AASIST-L | ~85K parameters | small, from-scratch | [7] |
| AASIST | ~297K parameters | small, from-scratch (reference) | [7] |
| Distilled WavLM student | ~1M parameters (target) | small, pretrained | [8], distilled from row 5 |
| WavLM + AASIST back-end, LoRA-adapted | ~94M parameters | large, pretrained | [7, 8, 9] |

The large system uses an AASIST back-end rather than a linear head because SSL front-end plus AASIST back-end is the architecture that current published and deployed systems actually use [21, 23]. On cached frozen features the back-end is inexpensive.

Parameter counts are indicative. Actual counts, MACs per second of audio, and measured latency are reported from the frozen configurations.

### Distillation protocol

The student is distilled from the frozen large system on **clean training data only**. No channel-condition data enters distillation, so the student cannot acquire channel robustness by having seen the channel — any robustness it shows must be inherited from the representation. Distillation loss, student architecture, temperature, and stopping criterion are frozen before channel scoring.

The student's target size is declared in advance so that it is not tuned to produce a favourable contrast. If the student fails to reach an accuracy bar on clean data, it is reported as a failed distillation and H1 is reported as untestable rather than being rescued by re-tuning.

### Training controls

Every detector receives identical train/development/test source partitions. Hyperparameters are selected using only clean training/development data and then frozen. Channel-condition test scores cannot be used to tune architectures, training schedules, thresholds, or preprocessing. Three independent training seeds are used for trainable neural systems if compute permits.

## 7. Evaluation protocol

### Primary condition

For detector `m`:

```text
EER_G711(m)  = mean(EER_C2(m), EER_C3(m))
ΔEER_G711(m) = EER_G711(m) − EER_C0(m)
```

### Primary analysis — two difference-in-differences contrasts

```text
DiD_pretraining = ΔEER_G711(AASIST-L)        − ΔEER_G711(distilled student)
DiD_size        = ΔEER_G711(distilled student) − ΔEER_G711(WavLM + AASIST)
```

Both detectors in each contrast are evaluated on the same paired recordings, so each contrast is estimated with a paired cluster bootstrap whose resampling unit is the original source recording. All channel versions of one source, and all detectors' scores for that source, move together in every replicate. This is what makes the contrasts well powered despite the small number of systems — the sample size is the number of recordings, not the number of models.

H1 is supported if `DiD_pretraining` excludes zero from above. H2 is supported if `DiD_size` excludes zero from above.

A consistency check reports whether `DiD_pretraining + DiD_size` is compatible with the confounded diagonal contrast within bootstrap uncertainty. Material disagreement indicates interaction between the two factors and is reported rather than suppressed.

Where repeated speech from the same speaker creates dependence, a speaker-cluster sensitivity analysis is also reported.

### Bandwidth attribution — the original contribution

```text
Bandwidth effect        = EER_C1 − EER_C0
Additional μ-law effect = EER_C2 − EER_C1
Additional A-law effect = EER_C3 − EER_C1
```

Reported per system, so that H1 and H2 can be examined separately for the bandwidth and companding components. Since no published work separates these, this decomposition is reported as a primary result in its own right regardless of how the DiD contrasts resolve.

### Threshold-transfer check — secondary, not claimed

Following the recommendation of [19], each detector's operating threshold is selected once on the clean development set and transferred unchanged to C0–C3, reporting false-positive rate, false-negative rate, balanced accuracy, and change relative to C0. This is a deployment sanity check on the present systems, not a novel contribution: the argument and its strongest evidence belong to [19] and are cited as such.

### Compute measurement

Measured and reported for every detector: parameter count, model file size, MACs per second of audio, real-time factor on the declared target device, and peak resident memory under sustained inference, plus the same figures on the development workstation. Measurement conditions — device, thermal state, batch size, input length, repetitions, and how median and spread are computed — are preregistered.

### Secondary metrics

AUROC, AUPRC, EER separately for C2 and C3, and per-generator and per-speaker breakdowns where sample size and privacy permit.

## 8. Leakage and confounding controls

- Apply C1–C3 symmetrically to genuine and spoof samples.
- Use identical containers, input rates, duration handling, and file naming across classes.
- Use one pinned resampler configuration everywhere.
- Distil on clean data only, so the student cannot see the channel.
- Do not encode only one class or generator family.
- Do not allow channel labels to correlate with class labels.
- Freeze all thresholds before opening transformed test results.
- Store source IDs separately from human-readable identities.
- Audit duplicate or near-duplicate audio before splitting.
- Report exclusions and complete sample counts in every condition.

## 9. Decision rules

The project succeeds scientifically whether the hypotheses are supported or rejected, provided the protocol completes without leakage and uncertainty is reported.

Before final scoring, preregister:

1. the immutable test manifest;
2. the five detector configurations, including the student's target size;
3. the distillation protocol;
4. the seed policy;
5. C0–C3 transformation commands and the pinned resampler;
6. the primary G.711 macro-average and both DiD contrasts;
7. bootstrap units and replicate count;
8. the target device and the H5 budgets; and
9. handling of failed or corrupted samples.

Interpretation:

- `DiD_pretraining > 0` with interval excluding zero supports pretraining as a source of channel robustness.
- `DiD_size > 0` with interval excluding zero supports capacity as a separate source.
- Both positive: the factors contribute separately and additively.
- Pretraining positive, size not: **distillation is a viable route to edge deployment** — the most actionable outcome.
- Size positive, pretraining not: capacity is irreducible and edge deployment is genuinely constrained.
- Neither: this channel does not discriminate between the systems, reported as such.
- Similar C1, C2 and C3 results indicate bandwidth, not companding, is the dominant factor.
- Wide intervals produce an inconclusive result, not evidence of no effect.

## 10. Limitations

- Simulated G.711 is not a complete telephone network, and the channel protocol replicates published Thai telephony work [4, 12].
- The design has three cells, not four; no large from-scratch system anchors the missing cell.
- The pretraining contrast compares a distilled student against a from-scratch model of similar but not identical size; residual size difference is reported.
- Distillation quality is a confound: a weak student could fail for reasons unrelated to pretraining, which is why a clean-data accuracy bar is preregistered.
- Dataset access and Thai speaker diversity may constrain external validity.
- EER does not represent real scam prevalence or asymmetric deployment costs.
- On-device timings are specific to one device, one runtime, and one thermal environment.
- Results cannot establish that any detector is safe for forensic or financial decisions.

## 11. Deliverables

- Versioned source and transformed manifests
- Reproducible C0–C3 transformation script and environment
- Frozen detector configurations, distillation recipe, and model revisions
- Source-level prediction files
- EER, threshold-transfer, DiD and bootstrap analysis scripts
- Bandwidth-versus-companding decomposition tables
- On-device benchmark script and measured latency/memory tables
- Leakage audit, model card, ethics statement, and limitations

See [`feasibility.md`](feasibility.md) for schedule, compute and hardware plans, and [`compute-request.md`](compute-request.md) for the resource estimate.
