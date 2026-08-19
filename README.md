# ScaleGap-TH

**English** | [ภาษาไทย](README.th.md)

**Does the accuracy–compute frontier for Thai audio-deepfake detection get steeper over the telephone channel?**

ScaleGap-TH is a focused YSC Thailand research proposal. It measures how the accuracy cost of shrinking a Thai voice-spoofing detector changes when the audio arrives through a G.711 telephone channel instead of a clean microphone, and whether a detector small enough to run on a phone or a single-board computer still works under that channel.

## Research question

> Does G.711 telephone coding degrade small-capacity Thai audio-deepfake detectors more than large-capacity ones?

Thai voice-cloning scams arrive over the telephone, and the telephone path is narrowband: G.711 low-pass filters the signal to roughly 3.4 kHz. Most vocoder and TTS artifacts concentrate above that band. A high-capacity detector may retain redundant cues below 3.4 kHz; a small detector may have spent its entire capacity on the band that the channel deletes. If that is true, the standard deployment advice — use a small model at the edge — fails precisely where it is needed.

### Why this is not already answered

Prior work establishes each half of the question separately, but not the interaction:

- **Benchmark-to-real gap.** Müller et al. reported RawNet2 rising from 3.154% EER on ASVspoof 2019 LA to 37.819% on In-the-Wild [1]. This measures domain shift, not capacity.
- **Codec robustness.** The ADD-C benchmark evaluates six codecs — AMR-WB, EVS, IVAS, Opus, Speex-WB and SILK [2]. All are wideband; narrowband G.711 is not among them.
- **Real telephony conditions.** ASVspoof 2021 LA does include real a-law and μ-law transmission [10], but for English, with no capacity axis and no bandwidth-only control to separate low-pass filtering from companding.
- **Efficient anti-spoofing.** AASIST-L reaches competitive accuracy at roughly 85K parameters [7], and later work sparsifies the AASIST backend further for deployment [11]. These report efficiency on clean conditions, not efficiency under a channel.

No published work reports EER degradation as a function of detector capacity under narrowband telephone coding, and none separates bandwidth from companding while doing so.

## Hypotheses

- **H1 — frontier steepening (primary):** G.711 coding increases EER more for small-capacity detectors than for large-capacity detectors.
- **H2 — absolute degradation:** G.711 coding increases EER relative to paired clean audio for most detectors.
- **H3 — bandwidth attribution:** part of the degradation is explained by 8 kHz bandwidth reduction alone, with any remainder attributable to A-law/μ-law companding.
- **H4 — deployment viability:** at least one detector under 100K parameters retains usable G.711 performance while meeting a preregistered real-time factor and memory budget on the target device.

H1 is falsifiable and useful in both directions. If supported, small detectors cannot be deployed naively on phone audio. If rejected, cheap detectors are safe for this channel — a more surprising and equally actionable result.

## Experimental design

![ScaleGap-TH experimental design](docs/architecture.svg)

Editable source: [`docs/architecture.mmd`](docs/architecture.mmd)

Every test utterance produces four paired versions:

1. **C0 — Clean:** original 16 kHz PCM
2. **C1 — Bandwidth control:** 8 kHz linear PCM round trip, then resampled to the model input rate
3. **C2 — G.711 μ-law:** 8 kHz encode/decode round trip
4. **C3 — G.711 A-law:** 8 kHz encode/decode round trip

C1 separates the effect of telephone bandwidth from G.711 companding. Every transformation is applied identically to genuine and spoof audio.

## Detector panel — the capacity axis

Capacity is the manipulated variable, so the panel spans roughly three orders of magnitude using published systems only:

| System | Approx. scale | Role |
| --- | --- | --- |
| LFCC-GMM | non-neural | classical floor |
| AASIST-L | ~85K parameters | on-device candidate |
| AASIST | ~297K parameters | small neural reference |
| Frozen WavLM + linear head | ~94M frozen | large fixed representation |
| WavLM + single LoRA adapter | ~94M + adapters | large adapted |

Parameter counts are indicative. Final counts, multiply–accumulate operations, and on-device latency are **measured from the frozen configurations and reported**, not quoted from papers.

**This project does not propose a new architecture.** Designing a novel nano network is explicitly out of scope: AASIST-L already occupies that size class, and the contribution here is the frontier, not the model.

## Primary outcomes

For each detector, with `EER_G711 = mean(EER_C2, EER_C3)`:

```text
ΔEER_G711(m) = EER_G711(m) − EER_C0(m)
```

H1 is tested as a **difference-in-differences** between two detectors on the same paired recordings:

```text
DiD(small, large) = ΔEER_G711(small) − ΔEER_G711(large)
```

This contrast is well powered because the bootstrap resampling unit is the source recording, not the detector. Two contrasts are preregistered:

- **Range contrast (primary):** AASIST-L versus WavLM+LoRA — the largest capacity span, but capacity and architecture family are entangled.
- **Within-family contrast (secondary):** AASIST-L versus AASIST — architecture held constant, narrower capacity span.

The full five-system frontier is reported descriptively as a Pareto curve, not as a regression on capacity: with five systems, a slope estimate over capacity has low inferential power and will be presented as an effect estimate with uncertainty.

The study also reports:

- 95% paired cluster-bootstrap intervals over source recordings
- clean-to-channel threshold transfer without retuning on test data
- the bandwidth-versus-companding decomposition from C1
- measured parameters, MACs, real-time factor, peak memory and model file size on the target device

## Deployment arm

The smallest detector meeting the accuracy bar is exported and run on the declared target device with G.711-coded input, reporting real-time factor and peak resident memory under sustained load. This is a measurement, not a product: it establishes whether the frontier's small end is actually reachable on hardware a Thai deployment could afford.

## What this project will not claim

- It will not claim a novel architecture or a new state of the art on any benchmark.
- It will not claim that capacity alone explains the frontier — architecture family is entangled with size, which is why the within-family contrast is reported alongside the range contrast.
- It will not claim robustness to unseen voice generators.
- It will not claim that simulated G.711 is identical to a real Thai telephone network.
- It will not evaluate LINE, Discord, WhatsApp, replay, packet loss, or background noise.
- It will not claim forensic certainty from a detector score.
- It will not infer that the result generalizes to all Thai speakers or all future deepfakes.

## Repository contents

- [`docs/research-plan.md`](docs/research-plan.md) — hypotheses, controls, protocol, statistics, and limitations
- [`docs/feasibility.md`](docs/feasibility.md) — 12-week schedule, compute and hardware plan, and fallback paths
- [`docs/ethics.md`](docs/ethics.md) — consent, data governance, responsible release, and AI-tool disclosure
- [`docs/references.md`](docs/references.md) — research and official sources
- [`README.th.md`](README.th.md) — Thai-language overview

## Status

Research proposal and experimental design. Training code, datasets, predictions, and checkpoints are not included yet.

## License

MIT for original repository text and future code. External datasets, models, standards, and software retain their own licenses and access restrictions.
