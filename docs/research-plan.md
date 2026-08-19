# Research Plan: BenchmarkGap-TH

## 1. Problem statement

Audio-deepfake detectors can achieve low error on the benchmark used for development yet fail after deployment conditions change. Müller et al. reported that RawNet2 increased from 3.154% EER on ASVspoof 2019 LA to 37.819% on their In-the-Wild dataset; across evaluated configurations, they reported approximately 200–1,000% EER deterioration [1]. Shi et al. later evaluated three detector families under six communication codecs and five packet-loss rates and reported an average 5.30% degradation in EER from clean audio to codec-compressed audio with zero packet loss [2].

Those studies establish the general problem but do not answer a narrow Thai deployment question: does clean-benchmark performance predict performance after a reproducible G.711 telephone transformation?

### Research question

> Does detector performance on clean Thai audio-deepfake benchmark data predict performance after G.711 A-law and μ-law telephone coding?

### Hypotheses

- **H1 — degradation:** G.711 telephone coding increases EER relative to paired clean audio for most detector families.
- **H2 — ranking instability:** the ranking of detector families by clean EER is not perfectly preserved under G.711.
- **H3 — bandwidth contribution:** some degradation is explained by the 8 kHz bandwidth reduction alone, while additional degradation may be attributable to A-law/μ-law companding.

The study is descriptive and falsifiable. H1 is unsupported if paired intervals show no positive degradation. H2 is unsupported if rankings remain stable with high correlation and the clean winner consistently remains the G.711 winner. H3 is unsupported if G.711 conditions do not differ meaningfully from the 8 kHz linear-PCM control.

## 2. Contribution and scope

The contribution is a controlled, paired measurement of the clean-to-telephone benchmark gap for Thai audio-deepfake detection. It combines:

1. identical source recordings across all channel conditions;
2. symmetric transformation of genuine and spoof classes;
3. an 8 kHz bandwidth-only control;
4. four representative detector families;
5. fixed development thresholds transferred to channel conditions; and
6. paired uncertainty estimates and ranking-stability analysis.

### Explicit exclusions

The committed study does not investigate unseen generators, messaging applications, packet loss, replay, background noise, real telephone calls, partial deepfakes, calibration methods, abstention, Thai-tone auxiliaries, or a novel mixture-of-experts architecture. These may be discussed as future work but cannot enter the main experiment after preregistration.

## 3. Experimental conditions

Every test source produces four deterministic, paired conditions:

| ID | Condition | Purpose |
| --- | --- | --- |
| C0 | Original 16 kHz linear PCM | Clean benchmark reference |
| C1 | 8 kHz linear PCM round trip, then resampled to model rate | Isolate telephone-bandwidth reduction |
| C2 | G.711 μ-law encode/decode at 8 kHz, then resampled | Measure μ-law path |
| C3 | G.711 A-law encode/decode at 8 kHz, then resampled | Measure A-law path |

G.711 is formally defined by ITU-T Recommendation G.711 [3]. The exact encoder, decoder, resampler, software version, and command line will be frozen before final evaluation. Peak normalization, denoising, silence trimming, or loudness normalization will not be applied differently across conditions.

### Transformation integrity checks

- Each transformed file must map to exactly one source ID.
- Duration drift must remain within a preregistered tolerance.
- C1–C3 must contain the same source content and labels as C0.
- Hashes of transformation configuration and output manifests must be saved.
- Genuine and spoof counts must be identical across paired conditions.
- A random sample will be decoded and inspected for corruption before scoring.

## 4. Dataset design

Potential Thai sources include Chula Spoofed Speech (CSS) and SEA-Spoof, subject to current access approval and licenses [4–6]. The final dataset choice must be frozen before model comparison.

### Split controls

- Speaker-disjoint train, development, and test partitions
- Script/text-disjoint partitions where metadata permits
- Reference-audio-disjoint partitions for cloned speech
- Generator-checkpoint-disjoint partitions where metadata permits
- No transformed version of a test source in training or development
- One immutable test manifest used for every detector and condition

Generator identity is stratified where possible but is not a generalization target. The project will not claim robustness to unknown synthesis systems.

### Dataset fallback

If restricted Thai data is unavailable, the pipeline may be validated on lawfully accessible ASVspoof data. Such a fallback result must be labelled non-Thai and cannot answer the Thai research question. The report will distinguish a completed engineering validation from a completed Thai study.

## 5. Detector panel

The committed panel contains four systems representing different feature and adaptation strategies:

1. **LFCC-GMM** — classical acoustic-feature baseline
2. **AASIST** — raw-waveform spectro-temporal graph-attention detector [7]
3. **Frozen WavLM + classifier** — fixed self-supervised representation [8]
4. **WavLM + single LoRA adapter** — parameter-efficient adaptation [9]

Every detector receives identical train/development/test source partitions. Hyperparameters are selected using only the clean training/development data and then frozen. Channel-condition test scores cannot be used to tune architectures, training schedules, thresholds, or preprocessing.

Three independent training seeds will be used for trainable neural systems if compute permits. Seed-level results quantify optimization variability but are not treated as independent detector families in rank-correlation tests.

## 6. Evaluation protocol

### Primary condition

For detector `m`, define:

```text
EER_G711(m) = mean(EER_C2(m), EER_C3(m))
ΔEER_G711(m) = EER_G711(m) − EER_C0(m)
```

The macro-average prevents a decision based on whichever G.711 law happens to be easier.

### Primary analysis

For each detector, report `ΔEER_G711` with a 95% paired cluster-bootstrap confidence interval. The resampling unit is the original source recording; all channel versions of one source move together in every bootstrap replicate. Where repeated speech from the same speaker creates dependence, a speaker-cluster sensitivity analysis will also be reported.

### Bandwidth attribution

Compare:

```text
Bandwidth effect = EER_C1 − EER_C0
Additional μ-law effect = EER_C2 − EER_C1
Additional A-law effect = EER_C3 − EER_C1
```

This avoids attributing all narrowband degradation to companding.

### Threshold-transfer analysis

EER permits a new threshold for each condition, which can hide deployment failure. Therefore, each detector's operating threshold will also be selected once on the clean development set and transferred unchanged to C0–C3. Report:

- false-positive rate for genuine audio;
- false-negative rate for spoof audio;
- balanced accuracy; and
- change relative to C0.

### Ranking analysis

Across the four detector families, calculate Spearman rank correlation between C0 EER and G.711 macro-EER. Also report the fraction of paired bootstrap replicates in which the clean winner remains the G.711 winner.

With only four detector families, rank correlation has low inferential power. It is an exploratory effect estimate, not a universal claim about all detectors. Exact permutations and bootstrap intervals will be reported; the result will not be reduced to a p-value alone.

### Secondary metrics

- AUROC
- AUPRC
- EER separately for C2 and C3
- Per-generator and per-speaker breakdowns where sample size and privacy permit
- Inference latency and peak memory as descriptive engineering results

## 7. Leakage and confounding controls

- Apply C1–C3 symmetrically to genuine and spoof samples.
- Use identical containers, input rates, duration handling, and file naming across classes.
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
2. the four detector configurations;
3. the seed policy;
4. C0–C3 transformation commands;
5. the primary G.711 macro-average;
6. bootstrap units and replicate count; and
7. handling of failed or corrupted samples.

Interpretation:

- A positive `ΔEER_G711` whose interval excludes zero supports degradation for that detector.
- A clean/G.711 rank correlation near one with stable winner identity supports benchmark ranking transfer within this limited panel.
- Weak or unstable rank agreement supports a benchmark-ranking gap.
- Similar C1, C2, and C3 results indicate bandwidth—not G.711 companding—is the dominant measured factor.
- Wide intervals produce an inconclusive result, not evidence of no effect.

## 9. Limitations

- Simulated G.711 is not a complete telephone network.
- Four detector families provide only a preliminary ranking analysis.
- Dataset access and Thai speaker diversity may constrain external validity.
- EER does not represent real scam prevalence or asymmetric deployment costs.
- Current generators may not represent future synthesis systems.
- Results cannot establish that any detector is safe for forensic or financial decisions.

## 10. Deliverables

- Versioned source and transformed manifests
- Reproducible C0–C3 transformation script and environment
- Frozen detector configurations and model revisions
- Source-level prediction files
- EER, threshold-transfer, and bootstrap analysis scripts
- Clean-versus-channel tables and plots
- Leakage audit, model card, ethics statement, and limitations

See [`feasibility.md`](feasibility.md) for the schedule and LANTA resource plan.
