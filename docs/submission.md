# YSC Submission Text

Draft text for the competition forms. The student researcher and adviser must review and approve every field before submission. All numbers and citations must also be checked against the primary sources.

## Wording rule that applies everywhere

Say **"G.711 telephone coding"**, **"telephone-band conditions"**, or **"simulated telephone channel"**. Never say **"real telephony"** or **"real phone calls"**.

The study applies offline codec transformations to audio files. Real calls also involve packet loss, jitter, transcoding between networks, handset characteristics, room acoustics, and automatic gain control. These factors are not studied here, so the wording must keep simulated conditions separate from real calls.

## Title

### Recommended

**English**

> Bandwidth or Companding? Attributing Telephone-Channel Degradation in Thai Audio Deepfake Detection

**Thai**

> แบนด์วิดท์หรือ companding: การแยกสาเหตุการเสื่อมประสิทธิภาพของระบบตรวจจับเสียงปลอมภาษาไทยบนช่องสัญญาณโทรศัพท์

The question form makes the attribution task clear instead of naming only the general subject area.

`companding` is left in English, which is common in Thai technical writing. The adviser may prefer a Thai gloss such as *การบีบอัดและขยายสัญญาณแบบลอการิทึม*; either is defensible, but use one consistently across every field.

### Alternatives

| Register | Title |
| --- | --- |
| Plainer, application-first | Measuring How Telephone Bandwidth Limits Thai Voice Deepfake Detection |
| More technical | Separating Bandwidth Limitation from Logarithmic Quantization in Telephone-Band Thai Anti-Spoofing |
| Punchier | Which Half of the Telephone Breaks Deepfake Detection? |

The last is good as a **poster headline** but too informal for the written entry.

## One-sentence topic statement

**English**

> This project measures whether the loss in Thai voice-deepfake detection over telephone channels is caused by bandwidth limitation or by G.711 companding, determines how much bandwidth detection can survive, and tests whether band-limited training recovers the loss.

**Thai**

> โครงงานนี้วัดว่าการเสื่อมประสิทธิภาพของการตรวจจับเสียงปลอมภาษาไทยบนช่องสัญญาณโทรศัพท์เกิดจากการจำกัดแบนด์วิดท์หรือจาก companding ของ G.711 หาว่าการตรวจจับทนการลดแบนด์วิดท์ได้มากเพียงใด และทดสอบว่าการฝึกด้วยเสียงที่ถูกจำกัดแบนด์วิดท์กู้ความสามารถกลับคืนมาได้หรือไม่

## Abstract

### English (~120 words)

> Voice-cloning scams in Thailand often reach victims by telephone. A conventional G.711 path carries a narrow voice-frequency band of roughly 300–3400 Hz, so some information used by detectors may be missing. Existing studies usually apply the complete codec and report one overall degradation value, combining bandwidth reduction with A-law or μ-law companding. This project estimates their contributions separately using a paired bandwidth-only control. It then varies the low-pass cutoff to measure when detection crosses preregistered failure thresholds and tests whether training with band-limited audio recovers some of the loss. The results will show whether model training is likely to help under the tested conditions or whether missing bandwidth is the larger limitation.

### Thai (~120 คำ)

> มิจฉาชีพที่ใช้เสียงปลอมในประเทศไทยมักติดต่อเหยื่อทางโทรศัพท์ ช่องสัญญาณ G.711 แบบทั่วไปส่งผ่านเสียงในย่านความถี่แคบประมาณ 300–3400 Hz ทำให้ข้อมูลบางส่วนที่ระบบตรวจจับอาจใช้หายไป งานวิจัยที่ผ่านมามักใช้โคเดกครบทั้งกระบวนการและรายงานค่าการเสื่อมรวมเพียงค่าเดียว ซึ่งรวมผลจากการลดแบนด์วิดท์และ companding แบบ A-law หรือ μ-law โครงงานนี้ประมาณผลของทั้งสองส่วนแยกกันด้วยเงื่อนไขควบคุมเฉพาะแบนด์วิดท์ จากนั้นปรับค่าความถี่ตัดเพื่อวัดว่าประสิทธิภาพข้ามเกณฑ์ล้มเหลวที่กำหนดไว้เมื่อใด และทดสอบว่าการฝึกด้วยเสียงที่จำกัดแบนด์วิดท์ช่วยกู้ประสิทธิภาพกลับมาได้บางส่วนหรือไม่ ผลการทดลองจะช่วยบอกว่าการปรับวิธีฝึกโมเดลมีแนวโน้มช่วยได้ หรือข้อจำกัดหลักมาจากแบนด์วิดท์ที่หายไป

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

Do not write that bandwidth is expected to dominate as though it were a finding. It is a prediction, and the project is designed so that the opposite result would be more interesting.

## Significance statement

> If bandwidth reduction contributes most of the degradation, improving the detector alone may not recover information removed by the channel. Wideband systems such as VoLTE and Opus-based VoIP preserve more frequency content. This project measures how detection changes as bandwidth is reduced, providing evidence that may help guide the design and evaluation of anti-fraud systems for Thai telephone services.

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
- [ ] Student can explain the C1 control and why it makes attribution possible
