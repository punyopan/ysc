# Research Plan: ChannelMoLEx-TH

## 1. Problem statement

Audio-deepfake detectors often perform well when the training and test data share generator fingerprints, speakers, scripts, sampling rates, and codecs. In deployment, Thai scam audio may be produced by an unseen generator and transmitted through an unknown telephone or messaging channel. The research goal is to separate evidence of synthesis from artifacts introduced by transmission.

### Main hypothesis

A WavLM detector with channel-conditioned Mixture-of-LoRA-Experts routing, adversarial channel disentanglement, and cross-channel consistency training will generalize better than RawNet2, AASIST, fully fine-tuned WavLM, single-LoRA WavLM, and standard MoLEx under unseen-generator and unseen-channel conditions.

### Proposed title

**ChannelMoLEx-TH: Channel-Robust Mixture-of-LoRA Experts for Generalizable Thai Audio Deepfake Detection**

---

## 2. Architecture

```text
Input waveform
    ↓
Resample/normalize without destroying forensic cues
    ↓
WavLM encoder
    ↓
LoRA-expert bank in each selected Transformer layer
    ↓
┌─────────────────────────┬──────────────────────────┐
│ Spoof representation    │ Channel representation   │
│ synthesis evidence      │ codec/bandwidth/noise    │
└─────────────────────────┴──────────────────────────┘
    ↓
Channel-conditioned Top-K router
    ↓
Attentive temporal pooling or LSTM backend
    ↓
Spoof probability + uncertainty
```

### WavLM backbone

WavLM provides pretrained acoustic, phonetic, speaker, prosodic, and environmental representations. Start with the backbone frozen, train the expert adapters and classifier, then compare against selective or complete fine-tuning.

### Mixture of LoRA Experts

Insert multiple trainable low-rank adapters in parallel with selected WavLM Transformer feed-forward layers. A learned router activates only the Top-K experts for each sample. The router receives acoustic hidden states and channel context.

Expert meanings must not be assigned manually. Specialization should be measured after training through activation analysis by generator, codec, style, and attack.

### Dual representations

1. **Channel representation:** bandwidth, codec family, compression severity, packet loss, replay, noise, and reverberation.
2. **Spoof representation:** synthesis artifacts that should remain relatively stable across transformations of the same source recording.

The classifier primarily consumes the spoof representation. The channel representation guides routing but must not become a shortcut for the class label.

### Optional extensions

- Multi-resolution STFT/LFCC forensic branch
- Frame-level partial-deepfake localization
- Thai lexical-tone and duration auxiliary tasks
- Open-set or conformal abstention head
- Continual insertion of new LoRA experts

---

## 3. Training objectives

A candidate objective is:

```text
L_total = L_spoof
        + λ_channel L_channel
        + λ_adv L_channel_adversarial
        + λ_consistency L_cross_channel
        + λ_router L_load_balance
        + λ_calibration L_calibration
```

### Spoof loss

Binary cross-entropy or focal loss for bona fide versus spoof speech. Sampling should be balanced by attack family, speaker, and channel rather than only by the global class count.

### Channel loss

Train the channel branch to identify known transformations or acoustic channel attributes. Use an `unknown` category or attribute-based labels where application identity is unavailable.

### Adversarial channel-invariance loss

Attach a channel classifier to the spoof representation through a gradient-reversal layer. The auxiliary classifier learns to identify channels while the spoof representation learns to suppress channel-specific shortcuts.

### Cross-channel consistency loss

Create several channel versions of the same underlying recording and encourage consistent spoof embeddings and predictions. Supervised contrastive learning can group all transformations of the same source.

### Router regularization

Use load balancing to prevent collapse into a small number of experts. Measure expert entropy and utilization rather than assuming that balanced routing is automatically optimal.

### Calibration

Evaluate Brier loss, temperature scaling, ensembles, conformal prediction, or selective classification. The detector should be able to abstain when evidence is insufficient.

---

## 4. Dataset design

Potential research datasets include Chula Spoofed Speech (CSS), SEA-Spoof Thai, and general anti-spoofing data such as ASVspoof. Full CSS and SEA-Spoof access is restricted and must be obtained before the project begins.

Add modern Thai spoof sources only where licensing and consent permit:

- VITS-family systems
- F5-TTS/flow-matching systems
- ThonburianTTS or research-compatible variants
- Neural-codec language models
- Continuous-latent systems such as the VoxCPM/JaiTTS architectural family
- Commercial black-box systems under permitted evaluation terms

### Leakage controls

- Speaker-disjoint splits
- Script/text-disjoint splits
- Reference-audio-disjoint splits
- Checkpoint-disjoint splits
- Generator-family-disjoint final tests
- Matching channel distributions for real and fake audio
- Matching sampling rate, file container, loudness, duration, and preprocessing

Every channel transformation must be applied symmetrically to genuine and spoof classes. Otherwise, the detector may learn that a codec or sampling rate implies `fake`.

---

## 5. Generator-family holdout

Suggested families:

1. **Two-stage:** Tacotron 2/FastPitch/FastSpeech plus neural vocoder.
2. **End-to-end latent-variable:** VITS and related architectures.
3. **Diffusion/flow matching:** F5-TTS, E2-TTS, Matcha-derived systems.
4. **Neural-codec language models:** autoregressive or parallel prediction over speech tokens.
5. **Continuous-latent diffusion-autoregressive:** hierarchical semantic/acoustic systems without a conventional external codec tokenizer.
6. **Commercial black boxes:** provider-level holdout where architecture is unknown.

For each family F:

```text
Train: all available families except F
Validation: development generators/checkpoints that do not overlap final tests
Test: F only, with unseen speakers, scripts, and reference recordings
```

Keep at least one final family untouched until the architecture and hyperparameters are frozen.

---

## 6. Channel robustness

### Reproducible channel simulation

- G.711 μ-law/A-law
- AMR-NB and AMR-WB
- Opus at several voice-oriented bitrates
- AAC-LC, MP3, and WebM/Opus
- Multiple encoding generations
- 8 kHz narrowband and 16 kHz wideband paths
- Packet loss, burst loss, jitter, and concealment
- Resampling, clipping, automatic gain variation
- Additive noise and reverberation
- Loudspeaker-to-microphone replay

### Actual transmission track

Transmit consented research recordings through controlled accounts or calls. Record:

- Application and version
- Device and OS
- Date
- Network type and approximate condition
- Input/output hardware
- Noise suppression, echo cancellation, and AGC state
- Received sampling rate and container

Do not claim a platform uses a specific codec without direct verification.

### Evaluation settings

1. Known generator, known channel
2. Unseen generator, known channel
3. Known generator, unseen channel
4. **Unseen generator, unseen channel**

Setting 4 is the principal measure of success.

---

## 7. Metrics

Report aggregate and disaggregated results:

- EER
- min-DCF
- AUROC and AUPRC
- Spoof recall at 1% and 0.1% false-positive rates
- False acceptance and genuine rejection at selected operating thresholds
- Expected Calibration Error
- Brier score
- Risk versus coverage under abstention
- Clean-to-channel performance degradation
- Inference latency, memory, and throughput

Break results down by generator family, individual generator, channel, speaker, age group, speaking style, duration, Thai-only text, and Thai-English code-switching.

---

## 8. Baselines and ablations

### Baselines

1. RawNet2
2. AASIST
3. Frozen WavLM plus classifier
4. Fully fine-tuned WavLM
5. WavLM plus one LoRA adapter
6. Standard WavLM/MoLEx
7. Proposed channel-aware WavLM/MoLEx

### Ablation matrix

| Variant | Channel branch | MoLEx | Adversarial invariance | Cross-channel consistency |
|---|---:|---:|---:|---:|
| Full model | ✓ | ✓ | ✓ | ✓ |
| No channel branch | — | ✓ | — | ✓ |
| Single LoRA | ✓ | — | ✓ | ✓ |
| No invariance | ✓ | ✓ | — | ✓ |
| No consistency | ✓ | ✓ | ✓ | — |
| Plain WavLM | — | — | — | — |

Improvement must hold under the unseen-generator × unseen-channel test. Gains limited to familiar channels may indicate channel memorization.

---

## 9. Thai-specific studies

### Tone and prosody

Investigate whether tone trajectories, syllable duration, pause placement, and Thai-English transitions improve cross-generator generalization. These are auxiliary forensic cues, not definitive proof: natural speakers exhibit substantial variation.

### Text conditions

- Pure Thai
- Thai-English code-switching
- Numerals and currencies
- Names and loanwords
- Formal versus conversational text

### Speaking styles

- Formal
- Casual
- Excited
- Low-energy or whispered speech where permitted
- Adolescent, adult, and elderly speakers

---

## 10. Milestones

1. Obtain dataset approvals and prepare a dataset governance statement.
2. Build leakage-resistant manifests and channel augmentation pipeline.
3. Reproduce RawNet2, AASIST, and WavLM baselines.
4. Reproduce standard MoLEx.
5. Add dual channel/spoof representations and channel-conditioned routing.
6. Run leave-one-generator-family-out experiments.
7. Run simulated channel experiments.
8. Freeze the system before untouched real-platform tests.
9. Add calibration, abstention, and error analysis.
10. Package reproducible code, manifests, model card, and limitations.

---

## 11. Success criteria

The project succeeds if the proposed method:

- Reduces error relative to standard MoLEx on held-out generator families
- Reduces clean-to-channel degradation
- Improves unseen-generator × unseen-channel performance
- Maintains calibrated confidence or useful abstention
- Demonstrates through ablation that gains come from channel-aware design
- Avoids speaker, script, codec, and file-format leakage

The system should be presented as one forensic signal, not as proof that an audio recording is genuine or fake.
