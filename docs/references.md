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
  *Note:* the LA evaluation set includes real telephony transmission — condition C3 traverses a PSTN path using a μ-law codec at 8 kHz, and further conditions traverse a PBX using a-law and G.722 among others. This is the closest existing answer to the channel half of this project's question: English, no capacity axis, no bandwidth-only control.

## Thai spoofing data

- **[4]** Urai, T., Boonsarngsuk, P., and Chuangsuwanich, E. **Thai Speech Spoofing Detection Dataset with Variations in Speaking Styles**. Interspeech 2025, pp. 5643–5647. Introduces the Chula Spoofed Speech (CSS) dataset.
  https://www.isca-archive.org/interspeech_2025/urai25_interspeech.html

- **[5]** Wu, J. et al. **SEA-Spoof: Bridging the Gap in Multilingual Audio Deepfake Detection for South-East Asia** (2025).
  https://arxiv.org/abs/2509.19865

- **[6]** SEA-Spoof Hugging Face dataset and access policy.
  https://huggingface.co/datasets/Jack-ppkdczgx/SEA-Spoof

- **[12]** Urai, T. **Advancing voice spoofing detection in Thai: a comprehensive dataset and performance analysis on speaking styles and channel effects**. Chulalongkorn University thesis, 2025.
  https://digital.car.chula.ac.th/chulaetd/75120/
  **Must be read in full during week 1.** The title indicates coverage of channel effects on Thai spoofing detection. If it already reports narrowband telephone conditions on CSS, this project's channel protocol is a replication and the capacity axis becomes the sole novel contribution. The proposal must state this accurately either way.

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

- References **[10]–[14]** were added when the project was rescoped from a pure channel-gap study to a capacity-versus-channel study. **They have not yet been read in full by the student researcher and must be before submission.** Parameter counts, codec lists, author lists, and dataset conditions quoted here are working values to be verified against the primary sources — several were compiled from secondary summaries.
- Verify the current CSS and SEA-Spoof licenses and access conditions before downloading or processing data.
- G.711 transformation software and the resampler must be validated against the standard and pinned by version.
- Access dates should be refreshed in the final YSC submission.
- The In-the-Wild and ADD-C statistics motivate this study but do not constitute Thai telephone-network evidence.
