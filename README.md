# ScaleGap-TH

**English** | [ภาษาไทย](README.th.md)

**Does Thai telephone-channel robustness come from model size or from self-supervised pretraining — and can a phone-sized detector inherit it?**

ScaleGap-TH is a focused YSC Thailand research proposal. It separates two factors that the audio-deepfake literature has left entangled, measures them under a controlled G.711 telephone channel on Thai speech, and checks whether the resulting small detector actually runs on affordable hardware.

## Research question

> Does Thai telephone-channel robustness in audio-deepfake detection come from model size or from self-supervised pretraining, and can a phone-sized detector inherit it?

### Why the question exists

On clean benchmark data, model size barely matters. AASIST reports 0.83% EER on ASVspoof 2019 LA at 297K parameters; AASIST-L reports 0.99% at 85K [7]. A 3.5× reduction costs 0.16 percentage points.

Away from the benchmark the picture inverts. Detectors below 1% EER in-domain reach 30–60% on in-the-wild corpora, and RawNet2 exceeds 40% on most out-of-distribution sets, while large self-supervised systems stay in the 7–9% range [20, 21].

**But those robust systems are large *because* they are self-supervised-pretrained on tens of thousands of hours.** Their robustness may come from the pretraining corpus rather than parameter count — and the effect is not monotone in size, since a 317M-parameter Wav2Vec2-AASIST does not automatically beat smaller systems on cross-dataset leaderboards [21].

Nobody has separated the two. The distinction decides whether edge deployment is possible:

- If robustness comes from **size**, the device is a hard ceiling.
- If it comes from **pretraining**, a small *distilled* detector can inherit it, and the ceiling largely disappears.

Thai voice-cloning scams arrive by telephone, where G.711 low-pass filters to ~3.4 kHz [3] — removing the band where most vocoder artifacts live. This project asks the size-versus-pretraining question under exactly that channel.

## What is already published

Stated up front so nothing here is overclaimed. Full detail in [`docs/research-plan.md`](docs/research-plan.md) §2.

- **Thai telephony evaluation exists.** The CSS dataset paper reports baselines "investigated in telephony scenarios" and supporting Thai anti-spoofing "in real-world telephony situations" [4], with the associated thesis extending to channel effects [12]. **This project's channel protocol is a replication and says so.**
- **Real telephony codec conditions exist** — ASVspoof 2021 LA includes PSTN and PBX transmission with μ-law and a-law, for English [10].
- **Threshold-transfer failure is established** — a 0.21%-EER detector at its source threshold rejects 78.7% of genuine in-the-wild speech while target EER reads 11.2% [19]. **This project does not claim that argument**; it reports transferred-threshold error as a secondary check, citing [19].

## What is claimed here

1. **Size versus pretraining, disentangled** — the primary contribution. No published work holds one constant while varying the other for channel robustness.
2. **Bandwidth versus companding decomposition** — the C1 control separates 3.4 kHz low-pass filtering from A-law/μ-law companding. Not done in any language.
3. **On-device measurement under channel conditions for Thai.**

## Design: three cells, two clean contrasts

|  | From-scratch | SSL-pretrained |
| --- | --- | --- |
| **Small** (<1M) | AASIST-L (~85K) | **Distilled WavLM student** (~1M) |
| **Large** (>90M) | *(no standard system exists)* | WavLM + AASIST back-end (~94M) |

- **Pretraining effect, size held ~constant:** AASIST-L vs distilled student
- **Size effect, pretraining held constant:** distilled student vs WavLM + AASIST

The diagonal is the confounded contrast the literature reports; here it serves as an internal consistency check. The empty lower-left cell is a stated limitation — no standard large from-scratch anti-spoofing system exists.

The student is **distilled on clean data only**, so it never sees the channel. Any robustness it shows must be inherited from the representation, not learned from exposure.

## Experimental design

![ScaleGap-TH experimental design](docs/architecture.svg)

Editable source: [`docs/architecture.mmd`](docs/architecture.mmd)

Every test utterance produces four paired versions:

1. **C0 — Clean:** original 16 kHz PCM
2. **C1 — Bandwidth control:** 8 kHz linear PCM round trip, then resampled to the model input rate
3. **C2 — G.711 μ-law:** 8 kHz encode/decode round trip
4. **C3 — G.711 A-law:** 8 kHz encode/decode round trip

C1 separates telephone bandwidth from G.711 companding. Every transformation is applied identically to genuine and spoof audio, through one pinned resampler.

## Hypotheses

- **H1 — pretraining effect (primary):** at matched small size, the distilled detector degrades less under G.711 than the from-scratch one.
- **H2 — size effect (primary):** at matched pretraining, the small student degrades more than the full-size SSL system.
- **H3 — absolute degradation** under G.711 for most detectors.
- **H4 — bandwidth attribution:** how much degradation is bandwidth versus companding.
- **H5 — deployment viability:** the student meets the preregistered latency and memory budgets on the target device.

Every outcome is informative. Pretraining matters and size doesn't → **distillation is a viable route to edge deployment**, the most actionable result. Size matters and pretraining doesn't → edge deployment is genuinely constrained. Both → the factors contribute separately.

## Primary outcomes

```text
ΔEER_G711(m)    = mean(EER_C2, EER_C3) − EER_C0
DiD_pretraining = ΔEER_G711(AASIST-L)         − ΔEER_G711(student)
DiD_size        = ΔEER_G711(student)          − ΔEER_G711(WavLM + AASIST)
```

Each contrast uses a paired cluster bootstrap over source recordings, so the sample size is the number of recordings, not the number of models — which is what makes these well powered despite the small panel.

## What this project will not claim

- It will not claim a novel architecture or a new state of the art.
- It will not claim the threshold-transfer argument, which belongs to [19].
- It will not claim to be the first Thai telephony evaluation, which is [4, 12].
- It will not claim robustness to unseen voice generators.
- It will not claim that simulated G.711 equals a real Thai telephone network.
- It will not evaluate LINE, Discord, WhatsApp, replay, packet loss, or background noise.
- It will not claim forensic certainty from a detector score.
- It will not infer that results generalize to all Thai speakers or all future deepfakes.

## Repository contents

- [`docs/research-plan.md`](docs/research-plan.md) — novelty position, design, hypotheses, protocol, statistics, limitations
- [`docs/feasibility.md`](docs/feasibility.md) — 12-week schedule, compute and hardware plans, fallbacks
- [`docs/compute-request.md`](docs/compute-request.md) — preliminary resource estimate and allocation request
- [`docs/ethics.md`](docs/ethics.md) — consent, data governance, responsible release, AI-tool disclosure
- [`docs/references.md`](docs/references.md) — research and official sources
- [`README.th.md`](README.th.md) — Thai-language overview

## Status

Research proposal and experimental design. Training code, datasets, predictions, and checkpoints are not included yet.

## License

MIT for original repository text and future code. External datasets, models, standards, and software retain their own licenses and access restrictions.
