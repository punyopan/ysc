# YSC Submission Text

Ready-to-adapt text for competition forms. **Every field must be reviewed and approved by the student researcher and adviser before submission** — this is drafting support, not a finished entry. Verify all numbers and citations against primary sources first.

## Wording rule that applies everywhere

Say **"G.711 telephone coding"**, **"telephone-band conditions"**, or **"simulated telephone channel"**. Never say **"real telephony"** or **"real phone calls"**.

The study applies offline codec transformations to audio files. Real calls additionally involve packet loss, jitter, transcoding between networks, handset microphone characteristics, room acoustics, and automatic gain control — none of which are studied. A judge who notices that gap in a form field will doubt the rest of the entry, and the distinction is easy to keep.

## Title

### Recommended

**English**

> Bandwidth or Companding? Attributing Telephone-Channel Degradation in Thai Audio Deepfake Detection

**Thai**

> แบนด์วิดท์หรือ companding: การแยกสาเหตุการเสื่อมประสิทธิภาพของระบบตรวจจับเสียงปลอมภาษาไทยบนช่องสัญญาณโทรศัพท์

The question form states the contribution — attribution — rather than the subject area, so a reader knows immediately what will be answered.

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

> Voice-cloning scams in Thailand arrive by telephone, where G.711 restricts audio to roughly 300–3400 Hz — removing the frequency range where most synthesis artifacts occur. Existing studies apply telephone codecs whole and report a single degradation figure, which combines two independent effects: bandwidth reduction, which deletes information permanently, and companding, which adds recoverable quantization noise. This project separates them using a paired control condition that removes bandwidth without companding, then sweeps the low-pass cutoff to locate the point where detection collapses, and finally tests whether training on band-limited audio recovers the loss. The distinction determines whether telephone-channel deepfake detection can be improved through training or is limited by the network itself.

### Thai (~120 คำ)

> มิจฉาชีพที่ใช้เสียงปลอมในประเทศไทยติดต่อเหยื่อผ่านโทรศัพท์ ซึ่ง G.711 จำกัดสัญญาณเสียงไว้ที่ราว 300–3400 Hz อันเป็นการตัดย่านความถี่ที่ร่องรอยของการสังเคราะห์เสียงส่วนใหญ่ปรากฏอยู่ งานวิจัยที่ผ่านมาใช้โคเดกโทรศัพท์ทั้งก้อนแล้วรายงานค่าการเสื่อมเพียงค่าเดียว ซึ่งรวมผลสองอย่างที่เป็นอิสระต่อกันไว้ด้วยกัน คือการลดแบนด์วิดท์ที่ลบสารสนเทศทิ้งอย่างถาวร และ companding ที่เพิ่ม quantization noise ซึ่งกู้คืนได้ โครงงานนี้แยกสองปัจจัยดังกล่าวด้วยเงื่อนไขควบคุมแบบจับคู่ที่ลดแบนด์วิดท์โดยไม่มี companding จากนั้นกวาดค่าความถี่ตัดเพื่อหาจุดที่การตรวจจับพัง และทดสอบว่าการฝึกด้วยเสียงที่ถูกจำกัดแบนด์วิดท์กู้ความสามารถกลับมาได้หรือไม่ ความแตกต่างนี้ชี้ว่าการตรวจจับเสียงปลอมบนช่องสัญญาณโทรศัพท์จะพัฒนาได้ด้วยการฝึกโมเดล หรือถูกจำกัดโดยตัวเครือข่ายเอง

## Category

**Computer Science** (วิทยาการคอมพิวเตอร์) is the natural fit — the core is a controlled measurement study.

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

> If bandwidth reduction dominates, the evidence required to detect voice fraud is being destroyed by the telephone network before detection is attempted, and no amount of model training recovers it. Wideband codecs such as VoLTE and Opus-based VoIP preserve that frequency range. This project quantifies how much detection capability narrowband call infrastructure discards, which is information relevant to how anti-fraud systems for Thai telephone networks should be designed.

## Scope limitations to state in the entry

Including these strengthens rather than weakens the submission — they demonstrate that the design is understood.

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
