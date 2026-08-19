# BandGap-TH

**English** | [ภาษาไทย](README.th.md)

**How much does telephone bandwidth limit Thai voice-deepfake detection, and when does detection begin to fail?**

BandGap-TH is a YSC Thailand research proposal. It separates two effects of telephone coding such as bandwidth and companding, measures how detection changes as audio bandwidth is reduced, and tests whether training with band-limited audio can recover some of the lost performance.

## The problem

Thai voice-cloning scams often reach victims by telephone. A conventional G.711 telephone path carries a narrow voice-frequency band of roughly 300–3400 Hz [3]. Some artifacts used to distinguish synthetic speech may lie outside that range, so a detector trained on clean wideband audio may rely on information that is missing from telephone audio.

Published work shows that codecs can degrade detection [2, 10], and Thai telephony evaluations already exist [4, 12]. However, these studies do not clearly separate two effects that occur together in a G.711 path:

| | What it does | Consequence |
| --- | --- | --- |
| **Bandwidth reduction** | Removes frequencies above roughly 3.4 kHz | Information in the removed band is no longer available to the detector |
| **Companding** | Requantizes 16-bit linear audio to 8-bit logarithmic A-law or μ-law | Information remains, but the signal is changed by quantization |

Studies usually apply the complete codec and report one overall result. Measuring the two contributions separately can show whether model training is likely to help or whether missing bandwidth is the larger limitation.

## Research questions

1. **Attribution:** Of the EER increase caused by simulated G.711 coding on Thai audio, how much is associated with bandwidth reduction and how much with companding?
2. **Dose-response:** How does detection change as the low-pass cutoff is reduced, and at what cutoff does performance cross the preregistered failure thresholds?
3. **Mitigation:** Does training with band-limited audio recover some of the loss, and by how much?

## Hypotheses

- **H1 — attribution:** bandwidth contributes more than companding.
- **H2 — dose–response:** EER rises monotonically as cutoff falls, with an identifiable collapse threshold.
- **H3 — mitigation:** band-limited augmentation reduces `ΔEER` relative to an identical clean-trained baseline.

The hypotheses can be supported or rejected. If companding contributes more than expected, channel-aware training may have more room to help. If augmentation provides little or no recovery, the result would be consistent with information loss from bandwidth reduction, although other explanations would still need to be considered.

## Design

![BandGap-TH experimental design](docs/architecture.svg)

Editable source: [`docs/architecture.mmd`](docs/architecture.mmd)

**Decomposition set:** every test source produces four paired conditions:

| ID | Condition | Contains |
| --- | --- | --- |
| C0 | Clean 16 kHz linear PCM | reference |
| C1 | 8 kHz **linear PCM** round trip | bandwidth only |
| C2 | G.711 μ-law | bandwidth + companding |
| C3 | G.711 A-law | bandwidth + companding |

C1 is the bandwidth-only control. It removes the high-frequency band while retaining 16-bit linear precision. Comparing C1 with C2 and C3 estimates the additional change associated with companding after bandwidth has already been reduced.

**Sweep set:** a no-filter 16 kHz reference followed by low-pass cutoffs at 6 KHz, 4 KHz, 3.4 KHz, 2.5 KHz, 1.5 KHz, and 800 Hz.

The sweep keeps the sample rate fixed at 16 kHz and changes only the filter cutoff. This avoids introducing resampling as another variable. The decomposition set still requires an 8 kHz round trip to represent G.711, but the sweep does not.

## Primary outcomes

```text
Bandwidth part  = EER_C1 − EER_C0
Companding part = mean(EER_C2, EER_C3) − EER_C1
Attribution gap = Bandwidth part − Companding part

Recovery = (ΔEER_baseline − ΔEER_augmented) / ΔEER_baseline
```

Each contrast uses a paired cluster bootstrap over source recordings. All conditions created from the same recording move together in each bootstrap sample.

The cutoff curve will include two preregistered failure thresholds: the cutoff where EER doubles and the cutoff where EER reaches 10%.

## Practical relevance

If bandwidth reduction contributes most of the degradation, improving the detector alone may not recover information removed by a narrowband channel. Wideband systems such as VoLTE and Opus-based VoIP preserve more frequency content. The cutoff curve will estimate how detection performance changes as that content is removed.

## Detector panel

| System | Scale | Role | Tier |
| --- | --- | --- | --- |
| LFCC-GMM | non-neural | classical floor | 1 |
| AASIST-L | ~85K | on-device candidate; mitigation subject | 1 |
| AASIST | ~297K | small neural reference | 1 |
| WavLM + AASIST back-end | ~94M | large SSL reference | 2, optional |

No new architecture is proposed. The panel exists to show findings are not an artifact of one model.

## What this project will not claim

- It will not claim to study **real telephony**. Real calls add packet loss, jitter, transcoding, handset characteristics, room acoustics and gain control — none are studied here. This is offline simulated G.711.
- It will not claim to be the first Thai telephony evaluation, which is [4, 12].
- It will not claim the threshold-transfer argument, which belongs to [19].
- It will not claim a novel architecture or a new state of the art.
- It will not claim robustness to unseen voice generators.
- It will not claim forensic certainty from a detector score.
- It will not infer that results generalize to all Thai speakers or all future deepfakes.

## Repository contents

- [`docs/research-plan.md`](docs/research-plan.md) — novelty position, conditions, hypotheses, statistics, limitations
- [`docs/feasibility.md`](docs/feasibility.md) — tiered 12-week schedule with go/no-go gates
- [`docs/submission.md`](docs/submission.md) — title, abstract, and form text in English and Thai
- [`docs/compute-request.md`](docs/compute-request.md) — preliminary resource estimate and allocation request
- [`docs/ethics.md`](docs/ethics.md) — consent, data governance, responsible release, AI-tool disclosure
- [`docs/references.md`](docs/references.md) — research and official sources
- [`README.th.md`](README.th.md) — Thai-language overview

## Status

Research proposal and experimental design. Training code, datasets, predictions, and checkpoints are not included yet.

## License

MIT for original repository text and future code. External datasets, models, standards, and software retain their own licenses and access restrictions.
