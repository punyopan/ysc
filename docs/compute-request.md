# Compute Resource Request — BandGap-TH

## Status of the numbers in this document

Every figure below is a preliminary estimate rather than a measurement. The estimates provide a bounded starting point for a resource request and should be treated only as order-of-magnitude values.

The week-4 compute pilot in [`feasibility.md`](feasibility.md) exists specifically to replace these estimates with measured values — peak GPU memory, examples per second, epoch time, and storage — before any full allocation is committed. Nothing in this file should be copied into the YSC proposal as a resource claim; the proposal reports measured pilot usage.

## Access route

ThaiSC requires the principal investigator for an educational-institution project to be a faculty adviser, so the student cannot hold the allocation directly. Two possible routes are:

1. **Join an existing project as a team member.** This is the preferred route. Usage would count against the host project's SHr budget, so the request below gives a bounded estimate.
2. **LANTA Proof of Concept.** This is a free trial with a 45-day validity period, a maximum of three team members, and an eligibility condition that no participant has registered for LANTA before. These restrictions may make it less suitable for this study.

Either way the adviser, not the student, is the PI.

## Billing model

On LANTA, **1 SHr = one compute node for one hour**, and **one GPU node-hour costs 3 SHr**. A GPU node carries four NVIDIA A100 GPUs, and billing is per node, not per GPU — so a job using one GPU on a node still pays for four.

## Working-set assumptions

Estimates assume a deliberately subsampled corpus, not the full source dataset:

| Partition | Source utterances |
| --- | --- |
| Train | ~40,000 |
| Development | ~5,000 |
| Test | ~10,000 |

Test sources are scored in four decomposition conditions and seven sweep conditions — eleven conditions per detector. Splits are speaker-disjoint as specified in [`research-plan.md`](research-plan.md).

## Estimated resource requirement

Costs are given per tier, because Tier 1 is the committed study and the later tiers are extensions with go/no-go gates.

| Component | Tier | Runs on | Est. GPU node-hours | Est. SHr |
| --- | --- | --- | --- | --- |
| Decomposition + sweep generation | 1 | CPU, trivially parallel | — | ~2 |
| LFCC-GMM baseline | 1 | CPU | — | ~5 |
| AASIST-L + AASIST, 3 seeds each | 1 | GPU, 4 jobs packed per node | ~20 | ~60 |
| Scoring all detectors across 11 conditions | 1 | GPU, inference only | ~4 | ~12 |
| **Tier 1 subtotal** | | | **~24** | **~79** |
| AASIST-L retrained with augmentation, 3 seeds | 2 | GPU, small model | ~8 | ~24 |
| Scoring the augmented model | 2 | GPU, inference only | ~2 | ~6 |
| **Tier 2 subtotal** | | | **~10** | **~30** |
| WavLM + AASIST back-end, 3 seeds | 3 | GPU, the only substantial training job | ~15 | ~45 |
| **Tier 3 subtotal** | | | **~15** | **~45** |
| **All tiers** | | | **~49** | **~154** |
| Contingency — first-time HPC use, failed jobs, restarts | | | ~2× | **~300** |

**Requested: approximately 300 SHr for all three tiers, or approximately 160 SHr for Tiers 1 and 2 alone.**

At the Thai government and education rate, this is approximately the size of one minimum LANTA project. The week-4 pilot will replace this estimate with measured usage.

Tier 1 is planned to run on a single consumer GPU. A LANTA allocation would shorten the work but is not required to begin the core study.

### Storage

The preliminary storage estimate is 40 GB. The sweep applies only to the test partition, so storage covers eleven conditions for about 10,000 test sources plus one clean copy of the training partition. The pilot must verify this estimate before the request is finalized.

## What drives the cost up

Three decisions account for most of the estimated budget:

1. **Corpus size.** The Chula Spoofed Speech dataset contains roughly 1.33 million utterances. Using all of it would raise storage and compute by roughly 25× without improving a design that needs speaker-disjoint paired splits rather than maximum volume.
2. **Seed count.** Three seeds triples the neural budget. If the host allocation is tight, the declared fallback is one seed for Tier 3, declared before final scoring rather than after.
3. **Node packing.** A GPU node bills as four A100s whether or not all four are used. Seed replicates and independent detector runs are packed onto a single node; running one job per node wastes roughly 75% of the charge.

The sweep adds inference conditions but no additional training runs, so its estimated cost is limited to several node-hours of scoring.

## Graceful degradation

The tier structure is the degradation plan. If the allocation is reduced:

1. Manifest generation and LFCC-GMM are CPU work and cannot be blocked.
2. Tier 1's attribution result and cutoff curve depend only on scoring frozen detectors, so they survive almost any compute reduction.
3. Tier 2 is one retraining run of an architecture already known to work.
4. Tier 3 is dropped first and reported as not attempted.

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
> I am a YSC entrant working with [adviser name] at [school] on BandGap-TH, a study of Thai audio-deepfake detection under simulated telephone-band conditions. A G.711 path combines bandwidth reduction with 8-bit logarithmic companding. Published work usually reports their combined effect, so this project estimates the two contributions separately. It also varies the low-pass cutoff to measure when detection crosses predefined failure thresholds and tests whether training with band-limited audio recovers some of the loss. The question matters in Thailand because voice-cloning scams often reach victims through telephone calls.
>
> I would like to ask whether I could run this as a team member under an existing project, rather than requesting a separate allocation.
>
> The preliminary request is approximately **300 SHr including contingency** for the full plan, or about **160 SHr** for the two committed tiers, with roughly **40 GB** of storage. The core study is designed to run on a single consumer GPU, while one optional configuration requires more substantial GPU training. A full breakdown is available at [repository link].
>
> These are preliminary estimates. I plan a pilot in week 4 to measure actual peak memory, throughput, and epoch time, and I will report measured figures back before scaling up. The study is tiered with explicit stopping points, so a tighter allocation simply means completing fewer tiers rather than failing.
>
> The project produces artifacts that may be reusable beyond it: a paired band-limited and G.711-transformed Thai spoofing evaluation set, a reproducible transformation pipeline, and source-level predictions across eleven conditions.
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
