# ChannelMoLEx-TH

**Channel-aware, generator-generalizable Thai voice-spoof detection**

This repository contains a research proposal for a Thai audio-deepfake detector designed to remain reliable when both the voice generator and transmission channel are unknown.

## Research question

> Can a channel-aware WavLM/Mixture-of-LoRA-Experts detector distinguish genuine Thai speech from synthetic speech produced by unseen generator families after transmission through real telephone and messaging codecs?

## Why this matters

Conventional anti-spoofing systems can achieve excellent in-dataset results while learning generator-, codec-, or dataset-specific fingerprints. Real scam audio may instead involve an unseen TTS model, 8 kHz telephony, Opus/AAC compression, packet loss, noise suppression, replay, or repeated transcoding.

The central objective is therefore **not the lowest error on familiar fakes**, but the smallest performance collapse under an **unseen generator × unseen channel** evaluation.

## Proposed architecture

```text
Thai waveform
    ↓
WavLM speech encoder
    ↓
Mixture of LoRA Experts in Transformer layers
    ├── spoof-artifact experts
    ├── neural-codec experts
    ├── telephone/channel experts
    └── prosody/Thai-tone experts
    ↓
Separated spoof and channel representations
    ↓
Channel-conditioned Top-K expert routing
    ↓
Real/spoof score + calibrated uncertainty
```

The spoof representation is trained to remain stable across channels, while a separate channel embedding guides expert routing. This allows the detector to adapt to bandwidth, compression, packet loss, replay, and noise without treating the channel itself as evidence of spoofing.

## Evaluation protocol

### Leave-one-generator-family-out

Generator families are held out completely during training:

1. Two-stage acoustic model + vocoder
2. End-to-end VITS-family models
3. Diffusion/flow-matching models
4. Neural-codec language models
5. Continuous-latent diffusion-autoregressive models
6. Commercial black-box providers

### Channel tracks

**Reproducible simulation:** G.711, AMR-NB/WB, Opus, AAC, MP3, packet loss, jitter, resampling, clipping, noise, reverberation, and replay.

**Real transmission:** controlled telephone, LINE, Discord, WhatsApp/Signal, or WebRTC tests with platform version, device, date, and processing settings documented.

### Headline condition

```text
Unseen Thai voice generator + unseen transmission channel
```

## Metrics

- Equal Error Rate (EER)
- minimum Detection Cost Function (min-DCF)
- AUROC
- Spoof recall at low false-positive rates
- Calibration error and Brier score
- Risk-versus-coverage under abstention
- Performance degradation relative to clean audio
- Latency, memory, and real-time factor

## Repository contents

- [`docs/research-plan.md`](docs/research-plan.md) — complete methodology and experiment design
- [`docs/references.md`](docs/references.md) — primary research sources

## Intended outcome

A successful project should demonstrate improved robustness on unseen Thai generators and communication channels, provide calibrated uncertainty, and avoid channel-label leakage. It should not claim forensic certainty from a single classifier score.

## Status

Research proposal and experimental design. No detector checkpoint is included yet.

## License

MIT for original repository text and future code. External datasets and models retain their own licenses and access restrictions.
