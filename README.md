# BandGap-TH

**English** | [ภาษาไทย](README.th.md)

**Phone lines throw away most of the audio. Do they also throw away the evidence that a Thai voice was faked?**

BandGap-TH is a research proposal for YSC Thailand. A telephone codec damages audio in two ways at once, and this project pulls the two apart to find out which one actually breaks deepfake detection. It then narrows the audio band step by step to locate the point where detection gives out, and checks whether retraining on narrowed audio buys any of it back.

## The problem

Voice-cloning scams in Thailand come in over the phone. A phone line is narrow: G.711 keeps only about 300 to 3400 Hz [3]. The giveaways that vocoders and TTS systems leave behind mostly sit above that. So a detector trained on clean, full-bandwidth audio may be leaning on evidence that never reaches the far end of the call.

That codecs hurt detection is known [2, 10], and Thai telephony has been evaluated before [4, 12]. What no one has pulled apart is which half of the codec causes it. A telephone codec is really two operations running at the same time:

| | What it does | Consequence |
| --- | --- | --- |
| **Bandwidth reduction** | Filters out everything above roughly 3.4 kHz | The evidence is gone for good, and no amount of training brings it back |
| **Companding** | Requantizes 16-bit linear to 8-bit logarithmic (A-law/μ-law) | The evidence is still there, buried in quantization noise, so it can be recovered |

Every study so far runs the codec end to end and reports one number, which is the two effects added together. Knowing which of them is larger decides whether better training can fix this, or whether the network itself sets the ceiling.

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

**Sweep set:** a no-filter 16 kHz reference followed by low-pass cutoffs at 6000, 4000, 3400, 2500, 1500, and 800 Hz.

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
