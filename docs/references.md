# References

Numbers are stable citation keys used throughout the other documents. They are written as explicit labels rather than an auto-numbered list so that the keys do not shift.

## Benchmark generalization and channel robustness

- **[1]** Müller, N. M., Czempin, P., Dieckmann, F., Froghyar, A., and Böttinger, K. **Does Audio Deepfake Detection Generalize?** (2024 revision).
  https://arxiv.org/abs/2203.16263

- **[2]** Shi, H. et al. **Benchmarking Audio Deepfake Detection Robustness in Real-world Communication Scenarios** (2026 revision). Introduces the ADD-C test set.
  https://arxiv.org/abs/2504.12423
  *Note:* ADD-C evaluates six codecs — AMR-WB, EVS, IVAS, Opus, Speex (WB) and SILK. All are wideband; narrowband G.711 is **not** among them. Cite this paper for "codecs degrade detection", not for "G.711 has been studied".

- **[3]** International Telecommunication Union. **ITU-T Recommendation G.711: Pulse Code Modulation (PCM) of Voice Frequencies**.
  https://www.itu.int/rec/T-REC-G.711/

- **[10]** ASVspoof 2021 Challenge. **Evaluation Plan**, and **ASVspoof 2021: accelerating progress in spoofed and deepfake speech detection**.
  https://www.asvspoof.org/asvspoof2021/asvspoof2021_evaluation_plan.pdf
  https://arxiv.org/abs/2210.02437
  *Note:* the LA evaluation set includes real telephony transmission — condition C3 traverses a PSTN path using a μ-law codec at 8 kHz, and further conditions traverse a PBX using a-law and G.722 among others. This is the closest existing answer to the channel half of this project's question: English, no separation of size from pretraining, no bandwidth-only control.

## Thai spoofing data

- **[4]** Urai, T., Boonsarngsuk, P., and Chuangsuwanich, E. **Thai Speech Spoofing Detection Dataset with Variations in Speaking Styles**. Interspeech 2025, pp. 5643–5647. Introduces the Chula Spoofed Speech (CSS) dataset, ~1.33M utterances, five TTS systems, varied speaking styles.
  https://www.isca-archive.org/interspeech_2025/urai25_interspeech.pdf
  **Critical for this project's novelty position.** The paper reports that its AASIST and RawNet2 baselines "were investigated in telephony scenarios", and that the dataset supports Thai anti-spoofing "in real-world telephony situations". Thai plus telephony plus small from-scratch detectors is therefore already published. This project's channel protocol replicates it; the contribution is the size-versus-pretraining disentanglement, which CSS does not vary (both its baselines are ~0.3M parameters and from-scratch).

- **[5]** Wu, J. et al. **SEA-Spoof: Bridging the Gap in Multilingual Audio Deepfake Detection for South-East Asia** (2025).
  https://arxiv.org/abs/2509.19865

- **[6]** SEA-Spoof Hugging Face dataset and access policy.
  https://huggingface.co/datasets/Jack-ppkdczgx/SEA-Spoof

- **[12]** Urai, T. **Advancing voice spoofing detection in Thai: a comprehensive dataset and performance analysis on speaking styles and channel effects**. Chulalongkorn University thesis, 2025.
  https://digital.car.chula.ac.th/chulaetd/75120/
  **Must be read in full during week 1.** The associated paper [4] already confirms telephony evaluation, so this thesis likely reports channel conditions in more detail. Read it to establish exactly which codecs and conditions are covered, then update the novelty position in `research-plan.md` §2. The channel protocol is already stated as a replication throughout; what this reading settles is whether anything beyond the size-versus-pretraining contrasts and the C1 decomposition remains open.

## Detectors and efficiency

- **[7]** Jung, J.-w. et al. **AASIST: Audio Anti-Spoofing Using Integrated Spectro-Temporal Graph Attention Networks** (2022). Source of both AASIST (~297K parameters) and the lightweight AASIST-L (~85K parameters).
  https://arxiv.org/abs/2110.01200

- **[8]** Chen, S. et al. **WavLM: Large-Scale Self-Supervised Pre-Training for Full Stack Speech Processing** (2021).
  https://arxiv.org/abs/2110.13900

- **[9]** Hu, E. J. et al. **LoRA: Low-Rank Adaptation of Large Language Models** (2021).
  https://arxiv.org/abs/2106.09685

- **[11]** **SpAArSIST: Sparsified AASIST for Efficient and Reliable Anti-Spoofing** (2026). Deployment-oriented redesign of the AASIST backend.
  https://arxiv.org/abs/2606.11674
  Relevant because it establishes that the sub-100K deployment size class is already actively optimized. This project therefore measures a frontier rather than proposing a new small architecture.

- **[13]** **Detecting Audio Deepfakes on the Edge: Lightweight SSL-Based Detection in a Browser Plugin** (2026).
  https://arxiv.org/abs/2606.30780

- **[14]** **A Lightweight and Efficient Model for Audio Anti-Spoofing**. ACM Multimedia Asia 2023.
  https://dl.acm.org/doi/abs/10.1145/3595916.3626403

## Generalization, deployment auditing, and leaderboards

- **[19]** **When EER Hides Deployment Failure: Auditing Threshold Transfer and Unlabeled Score Calibration for Speech Deepfake Detectors** (2026).
  https://arxiv.org/abs/2606.21584
  **This paper owns the threshold-transfer argument.** A 0.21%-EER detector operated at its source threshold rejects 78.7% of genuine In-the-Wild speech while its target EER still reads 11.2%. It also shows that any strictly increasing score transform — z-norm, temperature/shift calibration, embedding mean alignment under a frozen linear head — cannot change EER, and recommends reporting error at a transferred threshold alongside EER. ScaleGap-TH adopts that recommendation as a secondary check and does not claim the argument.

- **[20]** **How Well Do Current Speech Deepfake Detection Methods Generalize to the Real World?** (2026).
  https://arxiv.org/abs/2603.05852

- **[21]** **Speech DF Arena: A Leaderboard for Speech DeepFake Detection Models** (2025).
  https://arxiv.org/abs/2509.02859
  Source of the observation that larger is not automatically better — Wav2Vec2-AASIST at ~317M parameters does not necessarily outperform smaller systems across datasets, while lightweight models trade efficiency for higher EER. This non-monotonicity is why size and pretraining must be separated rather than treated as one axis.

- **[22]** **When Spoof Detectors Travel: Evaluation Across 66 Languages in the Low-Resource Language Spoofing Corpus** (2026).
  https://arxiv.org/abs/2603.02364
  Check during week 1 whether Thai is among the covered languages and what is reported for it.

- **[23]** **AASIST3: KAN-Enhanced AASIST Speech Deepfake Detection using SSL Features and Additional Regularization for the ASVspoof 2024 Challenge** (2024).
  https://arxiv.org/abs/2408.17352
  Representative of the current dominant pattern — self-supervised front-end plus graph-attention back-end — which is why the large system in this panel uses an AASIST back-end rather than a linear head.

## Competition and compute

- **[15]** Young Scientist Competition (YSC), NSTDA. **Competition requirements and research-ethics conditions**.
  https://www.nstda.or.th/ysc/requirements/

- **[16]** Young Scientist Competition (YSC), NSTDA. **Forms and manuals**.
  https://www.nstda.or.th/ysc/logo-and-forms-manuals/

- **[17]** NSTDA Supercomputer Center (ThaiSC). **LANTA Supercomputer system overview**.
  https://thaisc.io/thaisc-resorces/lanta

- **[18]** NSTDA Supercomputer Center (ThaiSC). **Services and LANTA access information**.
  https://thaisc.io/services

## Notes

- References **[10]–[14]** and **[19]–[23]** were added during two rescopes of this project. **They have not yet been read in full by the student researcher and must be before submission.** Parameter counts, EER figures, codec lists, author lists, and dataset conditions quoted here are working values compiled partly from secondary summaries, and must be verified against the primary sources.
- **[4], [12] and [19] are the three highest-priority reads.** They determine how much of this project is novel. Read them in week 1 and update the novelty position in `research-plan.md` §2 against what they actually report.
- Verify the current CSS and SEA-Spoof licenses and access conditions before downloading or processing data.
- G.711 transformation software and the resampler must be validated against the standard and pinned by version.
- Access dates should be refreshed in the final YSC submission.
- The In-the-Wild and ADD-C statistics motivate this study but do not constitute Thai telephone-network evidence.
