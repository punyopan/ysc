# YSC Submission Text

Draft text for the competition forms. The student researcher and adviser must review and approve every field before submission. All numbers and citations must also be checked against the primary sources.

## Wording rule that applies everywhere

Say **"G.711 telephone coding"**, **"telephone-band conditions"**, or **"simulated telephone channel"**. Never say **"real telephony"** or **"real phone calls"**.

The study applies offline codec transformations to audio files. Real calls also involve packet loss, jitter, transcoding between networks, handset characteristics, room acoustics, and automatic gain control. These factors are not studied here, so the wording must keep simulated conditions separate from real calls.

## Title

### Recommended

**English**

> When Does the Telephone Break Thai Voice-Deepfake Detection? Separating Bandwidth from Companding

**Thai**

> โทรศัพท์ทำให้การตรวจจับเสียงปลอมภาษาไทยล้มเหลวเมื่อใด: การแยกผลของแบนด์วิดท์ออกจาก companding

The first clause states what a reader outside the field cares about — whether detection survives a phone call — and the second states the method that makes the answer trustworthy. A title that opens with the attribution alone is accurate but reads to a non-specialist as a comparison between two kinds of signal distortion, which understates what the study is for.

`companding` is left in English, which is common in Thai technical writing. The adviser may prefer a Thai gloss such as *การบีบอัดและขยายสัญญาณแบบลอการิทึม*; either is defensible, but use one consistently across every field.

### Alternatives

| Register | Title |
| --- | --- |
| Method-first | Bandwidth or Companding? Attributing Telephone-Channel Degradation in Thai Audio Deepfake Detection |
| Plainer, application-first | Measuring How Telephone Bandwidth Limits Thai Voice Deepfake Detection |
| More technical | Separating Bandwidth Limitation from Logarithmic Quantization in Telephone-Band Thai Anti-Spoofing |
| Punchier | Which Half of the Telephone Breaks Deepfake Detection? |

The method-first form is the right choice if the entry is read mainly by speech-processing specialists. The last is good as a **poster headline** but too informal for the written entry.

## Headline sentence

Every field of the entry, the poster, and the spoken presentation should lead to one sentence of this shape. Fill in the measured values once Tier 1 completes; before then, present it as the question the study answers.

**English**

> Detection crosses its failure threshold below **X** kHz. A conventional Thai telephone path delivers 3.4 kHz. Therefore detectors of this class **are / are not** usable on telephone audio under the tested conditions.

**Thai**

> การตรวจจับข้ามเกณฑ์ล้มเหลวที่ความถี่ต่ำกว่า **X** kHz ขณะที่ช่องสัญญาณโทรศัพท์ไทยแบบทั่วไปส่งผ่านได้ 3.4 kHz ดังนั้นระบบตรวจจับกลุ่มนี้ **ใช้ได้ / ใช้ไม่ได้** กับเสียงโทรศัพท์ภายใต้เงื่อนไขที่ทดสอบ

The attribution, the recovery figure, and the bootstrap intervals are the evidence for that sentence, not competing headlines. State the sentence first and the evidence after it.

Keep the conditional wording. `X` is a measurement that does not exist until the experiment runs, and the sentence is scoped to the tested conditions rather than to real telephone calls.

## One-sentence topic statement

**English**

> This project measures whether the loss in Thai voice-deepfake detection over telephone channels is caused by bandwidth limitation or by G.711 companding, determines how much bandwidth detection can survive, and tests whether band-limited training recovers the loss.

**Thai**

> โครงงานนี้วัดว่าการเสื่อมประสิทธิภาพของการตรวจจับเสียงปลอมภาษาไทยบนช่องสัญญาณโทรศัพท์เกิดจากการจำกัดแบนด์วิดท์หรือจาก companding ของ G.711 หาว่าการตรวจจับทนการลดแบนด์วิดท์ได้มากเพียงใด และทดสอบว่าการฝึกด้วยเสียงที่ถูกจำกัดแบนด์วิดท์กู้ความสามารถกลับคืนมาได้หรือไม่

## Abstract

### English (~120 words)

> Voice-cloning scams in Thailand reach victims by telephone, but the detectors built to catch them are trained and benchmarked on clean wideband audio that no telephone call delivers. A conventional G.711 path carries roughly 300–3400 Hz, so information the detector may rely on can be missing before it ever sees the signal. Existing studies apply the complete codec and report one combined degradation value, mixing bandwidth reduction with A-law or μ-law companding. This project separates them using a paired bandwidth-only control, then varies the low-pass cutoff to find where detection crosses preregistered failure thresholds, and tests whether band-limited training recovers the loss. The separation matters practically: information altered by companding may be recoverable by training, while information removed by bandwidth cannot be, which decides whether a fix belongs in the model or in the transport.

### Thai (~120 คำ)

> มิจฉาชีพที่ใช้เสียงปลอมในประเทศไทยติดต่อเหยื่อทางโทรศัพท์ แต่ระบบตรวจจับที่สร้างขึ้นมารับมือกลับถูกฝึกและวัดผลด้วยเสียง wideband แบบสะอาดซึ่งไม่มีในสายโทรศัพท์จริง ช่องสัญญาณ G.711 แบบทั่วไปส่งผ่านเสียงในย่านประมาณ 300–3400 Hz ข้อมูลที่ระบบตรวจจับอาจใช้จึงอาจหายไปก่อนที่โมเดลจะได้รับสัญญาณด้วยซ้ำ งานวิจัยที่ผ่านมามักใช้โคเดกครบทั้งกระบวนการและรายงานค่าการเสื่อมรวมเพียงค่าเดียว ซึ่งรวมผลจากการลดแบนด์วิดท์เข้ากับ companding แบบ A-law หรือ μ-law โครงงานนี้แยกผลทั้งสองส่วนด้วยเงื่อนไขควบคุมเฉพาะแบนด์วิดท์ จากนั้นปรับค่าความถี่ตัดเพื่อหาจุดที่ประสิทธิภาพข้ามเกณฑ์ล้มเหลวที่กำหนดไว้ล่วงหน้า และทดสอบว่าการฝึกด้วยเสียงที่จำกัดแบนด์วิดท์กู้ประสิทธิภาพกลับมาได้หรือไม่ การแยกผลนี้มีความสำคัญเชิงปฏิบัติ เพราะข้อมูลที่ถูก companding เปลี่ยนแปลงอาจกู้คืนได้ด้วยการฝึกโมเดล แต่ข้อมูลที่ถูกตัดออกไปพร้อมแบนด์วิดท์กู้คืนไม่ได้ ซึ่งเป็นตัวชี้ว่าการแก้ไขควรเกิดที่ตัวโมเดลหรือที่ช่องสัญญาณ

## Category

**Computer Science** (วิทยาการคอมพิวเตอร์) is the best fit because the core project is a controlled evaluation of detection systems.

Engineering would be defensible only if Tier 3 completes and the entry leads with on-device deployment. Since Tier 3 is explicitly optional, do not choose a category that depends on it.

## Objectives

1. Determine what proportion of G.711 degradation in Thai audio-deepfake detection is attributable to bandwidth reduction versus companding.
2. Measure the relationship between low-pass cutoff frequency and detection error, and identify the cutoff at which detection collapses.
3. Measure how much of the loss band-limited training augmentation recovers, and what it costs on clean audio.

## Expected outcomes

State these as measurements to be made, not as results already known.

- A quantified attribution with confidence intervals: what fraction of telephone-channel degradation is bandwidth and what fraction is companding.
- A dose–response curve of detection error against low-pass cutoff, with two preregistered collapse thresholds.
- A recovery figure for band-limited augmentation, reported alongside its cost on clean audio.
- A reproducible transformation pipeline and paired evaluation set.
- A demonstration that replays one utterance at successive cutoffs against the measured curve, running offline from saved predictions.

Do not write that bandwidth is expected to dominate as though it were a finding. It is a prediction, and the project is designed so that the opposite result would be more interesting.

## Significance statement

> The common remedy for channel degradation is channel-aware augmentation: train the detector on degraded audio so it learns to cope. That remedy has room to work only if the damage comes from companding, which alters information still present in the signal. If the damage comes mostly from bandwidth reduction, the information has been removed and training cannot restore it — the fix would have to move to the transport, such as wideband VoLTE or Opus-based VoIP paths. This project measures which of the two dominates and at what cutoff detection fails, providing evidence that bears on whether anti-fraud work for Thai telephone services should invest in better detectors or in wider call bandwidth.

### If a reader says the answer is obvious

Anticipate the objection that bandwidth loss must obviously matter more, and answer it directly rather than in a footnote:

> If it were established, the field would not be investing in codec-aware augmentation as the primary remedy. The attribution has not been reported directly, so the size of each contribution is currently unknown, and the rejection of H1 would be the more consequential outcome because it would mean the loss is trainable. The study is designed so that either result is reportable.

## Scope limitations to state in the entry

These limitations should be stated directly so the entry does not imply a broader study than the protocol supports.

- The study uses offline simulated G.711 coding, not real telephone calls.
- Real calls additionally involve packet loss, jitter, transcoding, handset characteristics, room acoustics, and gain control, none of which are studied.
- Thai telephony evaluation of anti-spoofing models has been published previously; the channel conditions here replicate that work, and the contribution is the attribution, the cutoff sweep, and the mitigation measurement.
- Low-pass filtering models bandwidth limitation; it does not reproduce any specific network's frequency response.
- Results cannot establish that any detector is safe for forensic, banking, or law-enforcement decisions.

## AI-tool disclosure

Required by YSC rules. Adapt to the current official form; the version in [`ethics.md`](ethics.md) is the reference.

> Generative-AI tools were used for language editing, translation assistance, document organization, and code-review support. The student researcher verified the content, selected the methodology, conducted the experiments, analyzed the results, and takes responsibility for the final work.

## Pre-submission checklist

- [ ] Adviser has reviewed and approved the Thai title and abstract
- [ ] One rendering of `companding` chosen and used consistently in every field
- [ ] No field anywhere says "real telephony" or "real phone calls"
- [ ] Priority reads [4], [12], [19] completed and the novelty position updated
- [ ] Every number and citation verified against the primary source
- [ ] Category chosen without depending on optional Tier 3
- [ ] Expected outcomes written as measurements, not as predicted findings
- [ ] Scope limitations included rather than omitted
- [ ] AI-tool disclosure matches the current official form
- [ ] Student can explain the C1 control and why it makes attribution possible in one breath, without notes
- [ ] Headline sentence prepared, and every field leads toward it
- [ ] Answer to the "that is obvious" objection rehearsed
- [ ] Sweep demonstration runnable offline on the presentation laptop
