# BenchmarkGap-TH

**English** | [ภาษาไทย](README.th.md)

**Does clean-benchmark performance predict Thai audio-deepfake detection performance after G.711 telephone coding?**

BenchmarkGap-TH is a focused YSC Thailand research proposal. It measures whether detectors that rank well on clean Thai speech remain reliable—and retain the same ranking—after the same recordings pass through a controlled G.711 telephone channel.

## Research question

> Does detector performance on clean Thai audio-deepfake benchmark data predict performance after G.711 A-law and μ-law telephone coding?

Published work has found large benchmark-to-real-domain gaps. For example, RawNet2 increased from 3.154% EER on ASVspoof 2019 to 37.819% on an In-the-Wild dataset, while a communication benchmark reported an average 5.30% degradation in EER from clean audio to codec-compressed audio. This project tests a deliberately narrower version of that problem for Thai speech.

## Experimental design

![BenchmarkGap-TH experimental design](docs/architecture.svg)

Editable source: [`docs/architecture.mmd`](docs/architecture.mmd)

Every test utterance produces four paired versions:

1. **C0 — Clean:** original 16 kHz PCM
2. **C1 — Bandwidth control:** 8 kHz linear PCM round trip, then resampled to the model input rate
3. **C2 — G.711 μ-law:** 8 kHz encode/decode round trip
4. **C3 — G.711 A-law:** 8 kHz encode/decode round trip

C1 separates the effect of telephone bandwidth from G.711 companding. Every transformation is applied identically to genuine and spoof audio.

## Detector panel

The committed comparison contains four representative systems:

- LFCC-GMM
- AASIST
- frozen WavLM plus classifier
- WavLM plus one LoRA adapter

The project compares established detector families; it does not propose a large new architecture.

## Primary outcomes

For each detector, report:

```text
ΔEER(condition) = EER(condition) − EER(clean)
```

The primary condition is the macro-average of G.711 μ-law and A-law. The study also reports:

- 95% paired cluster-bootstrap intervals over source recordings
- clean-to-channel threshold transfer without retuning on test data
- Spearman rank correlation between clean and G.711 detector rankings
- the probability, under paired bootstrap resampling, that the clean winner remains the G.711 winner

Because the panel contains only four detector families, ranking statistics are exploratory and will be reported with uncertainty rather than treated as a high-powered universal estimate.

## What this project will not claim

- It will not claim robustness to unseen voice generators.
- It will not claim that simulated G.711 is identical to a real Thai telephone network.
- It will not evaluate LINE, Discord, WhatsApp, replay, packet loss, or background noise.
- It will not claim forensic certainty from a detector score.
- It will not infer that the result generalizes to all Thai speakers or all future deepfakes.

These exclusions are intentional: the project studies one question, **clean benchmark performance versus G.711 telephone performance**.

## Repository contents

- [`docs/research-plan.md`](docs/research-plan.md) — hypothesis, controls, protocol, statistics, and limitations
- [`docs/feasibility.md`](docs/feasibility.md) — 12-week schedule, LANTA plan, and fallback paths
- [`docs/ethics.md`](docs/ethics.md) — consent, data governance, responsible release, and AI-tool disclosure
- [`docs/references.md`](docs/references.md) — research and official sources
- [`README.th.md`](README.th.md) — Thai-language overview

## Status

Research proposal and experimental design. Training code, datasets, predictions, and checkpoints are not included yet.

## License

MIT for original repository text and future code. External datasets, models, standards, and software retain their own licenses and access restrictions.
