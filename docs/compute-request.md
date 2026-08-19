# Compute Resource Request — ScaleGap-TH

## Status of the numbers in this document

**Every figure below is a preliminary estimate, not a measurement.** They exist so that a resource request can name a specific, bounded number instead of asking open-endedly for GPUs. They are order-of-magnitude only.

The week-4 compute pilot in [`feasibility.md`](feasibility.md) exists specifically to replace these estimates with measured values — peak GPU memory, examples per second, epoch time, and storage — before any full allocation is committed. Nothing in this file should be copied into the YSC proposal as a resource claim; the proposal reports measured pilot usage.

## Access route

ThaiSC requires the Principal Investigator to be a faculty adviser for educational-institution projects, so a student cannot hold an allocation directly. Two routes exist:

1. **Join an existing project as a team member.** The normal route for this project. Usage draws down the host project's SHr budget, which is why the request below is stated as a bounded number.
2. **LANTA Proof of Concept.** Free trial, but a poor fit here: 45-day validity, maximum three team members, and it requires that no participant has registered for LANTA before.

Either way the adviser, not the student, is the PI.

## Billing model

On LANTA, **1 SHr = one compute node for one hour**, and **one GPU node-hour costs 3 SHr**. A GPU node carries four NVIDIA A100 GPUs, and billing is per node, not per GPU — so a job using one GPU on a node still pays for four.

## Working-set assumptions

The estimates assume a deliberately subsampled corpus, not the full source dataset:

| Partition | Source utterances |
| --- | --- |
| Train | ~40,000 |
| Development | ~5,000 |
| Test | ~10,000 |

Test sources are scored in all four conditions (C0–C3), giving ~40,000 scored files per detector. Splits are speaker-disjoint as specified in [`research-plan.md`](research-plan.md).

## Estimated resource requirement

| Component | Runs on | Est. GPU node-hours | Est. SHr |
| --- | --- | --- | --- |
| C0–C3 channel generation | CPU, trivially parallel | — | ~1 |
| LFCC-GMM baseline | CPU | — | ~5 |
| AASIST-L + AASIST, 3 seeds each | GPU, 4 jobs packed per node | ~20 | ~60 |
| WavLM + AASIST back-end, LoRA-adapted, 3 seeds — the distillation teacher | GPU, the only substantial training job | ~15 | ~45 |
| Teacher output caching over clean training data | GPU, inference only | ~2 | ~6 |
| Distilled student, 3 seeds, trained against cached teacher outputs | GPU, small model | ~8 | ~24 |
| Final C0–C3 scoring, all detectors | GPU, inference only | ~2 | ~6 |
| **Subtotal** | | **~47** | **~147** |
| Contingency — first-time HPC use, failed jobs, restarts | | ~2× | **~300–350** |

**Requested: approximately 350 SHr.**

At the Thai government/education rate this is on the order of a single minimum-size LANTA project. The request is small by design.

### Storage

Approximately 30 GB for the working set across four conditions, at 4 seconds and 16 kHz 16-bit per utterance, plus a WavLM feature cache and checkpoints. This is not a parallel-filesystem-scale requirement.

## What drives the cost up

Three decisions dominate the budget. Each is preregistered so the number cannot drift silently.

1. **Corpus size.** The Chula Spoofed Speech dataset contains roughly 1.33 million utterances. Using all of it would raise storage and compute by roughly 25× and would not improve the design, which needs speaker-disjoint paired splits rather than maximum volume. The subsample above is deliberate.
2. **Seed count.** Three seeds triples the neural budget. If the host allocation is tight, the declared fallback is one seed for the large WavLM system, declared before final scoring rather than after. The student keeps three seeds where possible, since distillation variance is a confound for the primary contrast.
3. **Node packing.** A GPU node bills as four A100s whether or not all four are used. Seed replicates and independent detector runs are packed onto a single node; running one job per node would waste roughly 75% of the charge.

## Graceful degradation

If the allocation is reduced, the project does not fail — it narrows, in this order:

1. Channel generation, LFCC-GMM, AASIST-L and AASIST complete on a single consumer GPU.
2. The large WavLM system is prioritized next, because it is both the distillation teacher and the large arm of the size contrast — both DiD contrasts depend on it.
3. The distilled student trains against cached teacher outputs, so it is cheap once the teacher exists.
4. If distillation fails its preregistered clean-data accuracy bar, both contrasts are reported as untestable, and the bandwidth-versus-companding decomposition — which does not depend on the student — still stands as a result.

Any detector that does not complete is reported as missing, never extrapolated.

## Data governance condition

**Settle this before uploading anything.** If restricted dataset audio is stored on LANTA under a host project, then the dataset license and the project's SRC/IRB approval must cover:

- that storage location;
- the host project's access list, including members who are not part of this study; and
- the retention and deletion schedule already required by [`ethics.md`](ethics.md).

Technical access to a shared allocation does not by itself grant permission to place restricted voice data there. If the licence cannot be shown to cover it, use the accessible-data fallback rather than proceeding.

## Draft cover note

To adapt before sending — replace the bracketed parts and have the adviser review it first.

> Dear [name],
>
> I am a YSC entrant working with [adviser name] at [school] on ScaleGap-TH, a study of Thai audio-deepfake detection under G.711 telephone coding. The question is whether channel robustness comes from model size or from self-supervised pretraining — the literature entangles the two, because the robust systems are large precisely because they are SSL-pretrained. If robustness comes from pretraining rather than size, a small distilled detector can inherit it and run on affordable hardware; if it comes from size, edge deployment is genuinely constrained. This matters for Thailand because voice-cloning scams arrive by telephone, on a narrowband channel.
>
> I would like to ask whether I could run this as a team member under an existing project, rather than requesting a separate allocation.
>
> The request is small and bounded: approximately **350 SHr including contingency**, and roughly **30 GB** of storage. Most of the work is CPU-only or runs on a single GPU; only one detector configuration requires substantial GPU training. A full breakdown is at [repository link].
>
> These are preliminary estimates. I plan a pilot in week 4 to measure actual peak memory, throughput, and epoch time, and I will report measured figures back before scaling up. If the allocation is tighter than this, the study degrades gracefully — the smaller detectors complete on modest hardware and still answer a reduced version of the question.
>
> The project produces artifacts that may be reusable beyond it: a paired G.711-transformed Thai spoofing evaluation set, a reproducible channel-transformation pipeline, and source-level predictions across four conditions.
>
> One question before anything is uploaded: if restricted dataset audio would be stored under the host project, I would need to confirm that the dataset licence and our ethics approval cover that location and the project's access list. I would rather resolve that in advance.
>
> Thank you for considering it.
>
> [name]

## Checklist before sending

- [ ] Adviser has agreed to act as PI or to be named as supervising adviser
- [ ] Estimates re-read and any that have changed updated
- [ ] Dataset access approval status stated honestly, including if still pending
- [ ] Storage location question raised explicitly
- [ ] Repository link included and the repository readable by the recipient
- [ ] Fallback plan stated, so a smaller allocation is still usable
