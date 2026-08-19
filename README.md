# BandGap-TH

**English** | [ภาษาไทย](README.th.md)

**Is the telephone destroying the evidence needed to detect Thai voice deepfakes — and how much bandwidth can you lose before detection fails?**

BandGap-TH is a YSC Thailand research proposal. It separates the two things a telephone codec does to audio, measures how much bandwidth Thai deepfake detection can survive, and tests whether training on band-limited audio wins the loss back.

## The problem

Thai voice-cloning scams arrive by telephone, and the telephone path is narrowband — G.711 restricts the signal to roughly 300–3400 Hz [3]. Most vocoder and TTS artifacts concentrate *above* that range. A detector trained on clean wideband audio may be relying on evidence the network deletes before the call arrives.

Published work shows codecs degrade detection [2, 10], and Thai telephony evaluation already exists [4, 12]. **What nobody separates is why.** A telephone codec does two independent things at once:

| | What it does | Consequence |
| --- | --- | --- |
| **Bandwidth reduction** | Low-pass filters away everything above ~3.4 kHz | The evidence is **deleted**. No training recovers it |
| **Companding** | Requantizes 16-bit linear → 8-bit logarithmic (A-law/μ-law) | The evidence **survives** under quantization noise. Recoverable |

Every published study applies the codec whole and reports one number — the sum of both. Which one dominates decides whether this problem is fixable in software or is a limit of the infrastructure.

## Research questions

1. **Attribution** — Of the EER increase caused by simulated G.711 coding on Thai audio, how much comes from bandwidth and how much from companding?
2. **Dose–response** — How much bandwidth can be removed before detection collapses, and at what cutoff?
3. **Mitigation** — Does training on band-limited audio recover the loss, and by how much?

Diagnosis, quantification, attempted remedy.

## Hypotheses

- **H1 — attribution:** bandwidth contributes more than companding.
- **H2 — dose–response:** EER rises monotonically as cutoff falls, with an identifiable collapse threshold.
- **H3 — mitigation:** band-limited augmentation reduces `ΔEER` relative to an identical clean-trained baseline.

Every outcome is informative. If companding turns out to dominate, the problem is far more tractable than expected — more surprising and more useful than the predicted answer. If augmentation recovers nothing, that is direct evidence the loss is informational, which strengthens the bandwidth interpretation.

## Design

![BandGap-TH experimental design](docs/architecture.svg)

Editable source: [`docs/architecture.mmd`](docs/architecture.mmd)

**Decomposition set** — every test source produces all four, paired:

| ID | Condition | Contains |
| --- | --- | --- |
| C0 | Clean 16 kHz linear PCM | reference |
| C1 | 8 kHz **linear PCM** round trip | bandwidth only |
| C2 | G.711 μ-law | bandwidth + companding |
| C3 | G.711 A-law | bandwidth + companding |

**C1 is the control that makes the whole project possible.** It strips the high band while keeping full 16-bit precision, so subtracting it isolates companding. Without C1 only the sum is observable — which is exactly why this has never been measured.

**Sweep set** — low-pass cutoff at 8000, 6000, 4000, 3400, 2500, 1500, 800 Hz.

The sweep **never changes sample rate**. Everything stays at 16 kHz and only the filter cutoff varies, so the resampler is removed from the experiment rather than merely controlled for. The decomposition set must round-trip through 8 kHz because G.711 requires it; the sweep does not, so it doesn't.

## Primary outcomes

```text
Bandwidth part  = EER_C1 − EER_C0
Companding part = mean(EER_C2, EER_C3) − EER_C1
Attribution gap = Bandwidth part − Companding part

Recovery = (ΔEER_baseline − ΔEER_augmented) / ΔEER_baseline
```

Each contrast uses a paired cluster bootstrap over source recordings — the sample size is the number of recordings, not the number of conditions, which is what makes these well powered.

Plus the cutoff curve with two preregistered collapse thresholds: where EER doubles, and where EER reaches 10%.

## Why it matters beyond the lab

If bandwidth dominates, the finding has an infrastructural consequence:

> **Narrowband telephony is actively destroying the evidence needed to detect voice fraud.**

Wideband codecs — VoLTE, Opus-based VoIP — preserve the band where the artifacts live. The cutoff curve quantifies exactly how much detection capability legacy narrowband call infrastructure throws away.

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
- [`docs/compute-request.md`](docs/compute-request.md) — preliminary resource estimate and allocation request
- [`docs/ethics.md`](docs/ethics.md) — consent, data governance, responsible release, AI-tool disclosure
- [`docs/references.md`](docs/references.md) — research and official sources
- [`README.th.md`](README.th.md) — Thai-language overview

## Status

Research proposal and experimental design. Training code, datasets, predictions, and checkpoints are not included yet.

## License

MIT for original repository text and future code. External datasets, models, standards, and software retain their own licenses and access restrictions.
