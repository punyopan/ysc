# BandGap-TH

**English** | [ภาษาไทย](README.th.md)

**Thai telephone networks carry 300–3400 Hz. Voice-deepfake detectors are trained on 16 kHz audio. This project measures whether detection still works after the network throws most of the signal away, and finds the cutoff where it stops working.**

Voice-cloning scams in Thailand reach victims by telephone. The detectors built to catch them are trained and benchmarked on clean wideband audio that no telephone call delivers. BandGap-TH measures how large that gap is, isolates what causes it, and tests whether it can be trained away.

The study is built to answer one question with a number:

> Detection crosses its failure threshold below **X** kHz. A conventional Thai telephone path delivers 3.4 kHz. Therefore detectors of this class **are / are not** usable on telephone audio under the tested conditions.

`X` is what the experiment measures. Everything else in this repository — the attribution, the cutoff curve, the recovery figure, the bootstrap intervals — is the evidence behind that sentence.

BandGap-TH is a YSC Thailand research proposal.

## The problem

A conventional G.711 telephone path carries a narrow voice-frequency band of roughly 300–3400 Hz [3]. Some artifacts used to distinguish synthetic speech may lie outside that range, so a detector trained on clean wideband audio may rely on information that is missing from telephone audio.

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

The design is symmetric: it is not built to confirm H1. Every hypothesis has a reportable outcome either way, and the thresholds are fixed before scoring so the goalposts cannot move once results arrive. If companding contributes more than expected, channel-aware training may have more room to help. If augmentation provides little or no recovery, the result would be consistent with information loss from bandwidth reduction, although other explanations would still need to be considered.

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

**C1 is the methodological contribution.** Published work applies the complete codec and reports one combined degradation number. C1 removes the high-frequency band while retaining 16-bit linear precision, so bandwidth reduction occurs without companding. Comparing C1 against C0 isolates the bandwidth effect; comparing C1 against C2 and C3 estimates the additional change associated with companding after bandwidth has already been reduced. Without this control the two causes cannot be separated.

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

## Why the answer matters

The common remedy for channel degradation is channel-aware augmentation: train the detector on degraded audio so it learns to cope. That remedy has room to work only if the damage comes from companding, because companding alters information that is still present in the signal. If the damage comes mostly from bandwidth reduction, the information has been removed and training cannot restore it.

| If damage is mostly… | Where a fix would have to act |
| --- | --- |
| **Companding** | The model. Channel-aware training and more robust representations have room to help |
| **Bandwidth** | The transport. Wideband paths such as VoLTE or Opus-based VoIP, because a removed band cannot be trained back |

That distinction bears on whether an anti-fraud effort should invest in better detectors or in wideband call paths, and we found no published study that reports it directly. The cutoff curve attaches a frequency to the question.

## Demonstration

The cutoff sweep doubles as the demonstration and is part of the committed work, not an optional extension. One Thai utterance is played at each cutoff in the sweep set — the 16 kHz reference, then 6 kHz, 4 kHz, 3.4 kHz, 2.5 kHz, 1.5 kHz and 800 Hz — while the detector score is shown against the measured curve, with the 3.4 kHz telephone band edge and the collapse thresholds marked on the frequency axis. It runs on a laptop from the Tier 1 prediction files and needs no GPU, no deployment toolchain, and no optional tier.

Presentation rules are in [`docs/feasibility.md`](docs/feasibility.md): show the score and the frozen operating threshold with its measured error rates, never a bare real/fake verdict, and use only project-owned or dataset-licensed audio.

## Detector panel

| System | Scale | Role | Tier |
| --- | --- | --- | --- |
| LFCC-GMM | non-neural | classical floor | 1 |
| AASIST-L | ~85K | on-device candidate; mitigation subject | 1 |
| AASIST | ~297K | small neural reference | 1 |
| WavLM + AASIST back-end | ~94M | large SSL reference | 2, optional |

No new architecture is proposed. The panel exists to show findings are not an artifact of one model.

## Scope boundaries

These boundaries are set in advance and frozen with the protocol, so the claims cannot expand after the results arrive.

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
