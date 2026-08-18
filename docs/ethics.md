# Ethics, Consent, and Responsible Use Plan

## Status

This is a pre-study governance plan, not an ethics approval. The student researcher and adviser must submit the current YSC forms and obtain every required SRC/IRB/institutional approval **before** collecting participant recordings or beginning any activity that requires prior review.

## Human-participant recordings

If new Thai speech is recorded, participation must be voluntary and based on understandable informed consent. The consent material should state:

- what speech will be recorded and for how long;
- whether a participant's voice may be used to create synthetic or voice-converted samples;
- the research purpose, foreseeable risks, and expected benefits;
- who can access raw and derived audio;
- how long recordings, embeddings, and metadata will be retained;
- whether any data or trained model will be released;
- how withdrawal works and what cannot be withdrawn after irreversible aggregation or publication; and
- contact information for the student, adviser, and reviewing body.

Minors require the approvals and assent/guardian-consent process specified by YSC and the responsible review body. No participant should be pressured by grades, authority relationships, or payment that obscures voluntary choice.

## Existing datasets and generated speech

For every dataset and generator, record the source, version, license, access approval, permitted purpose, and redistribution restrictions. Restricted audio must not be committed to GitHub. A model or provider's technical availability does not imply permission to clone a real person's voice.

Synthetic samples should use voices for which research use and synthesis are authorized. Do not impersonate public figures, teachers, classmates, or non-consenting people. If a provider prohibits security testing, automated generation, or redistribution, exclude it from the study.

## Data minimization and security

- Assign random participant identifiers; keep the identity key separate and encrypted.
- Collect only metadata needed for the stated analysis.
- Keep raw voice data and consent records in access-controlled storage, not in the public repository.
- Do not publish filenames or metadata that reveal identity.
- Define a retention/deletion date before collection.
- Share only aggregates or explicitly approved de-identified artifacts.
- Treat speaker embeddings as potentially identifying biometric data.
- Record who accessed restricted data and why.

The repository may contain code, schemas, synthetic examples authorized for release, and hashed manifests. It must not contain participant audio, credentials, platform tokens, identity keys, or restricted dataset files.

## Telephone and messaging-platform transmission

Real LINE, telephone, Discord, WhatsApp/Signal, or WebRTC transmission is a stretch experiment. It may begin only when:

1. the recording and transmission are covered by consent and required review;
2. controlled research accounts and consented devices are used;
3. current platform terms allow the activity;
4. no uninvolved person is recorded;
5. network/device/application versions and processing settings are logged; and
6. transferred audio is deleted according to the approved retention plan.

Simulation remains the primary reproducible channel test. If the conditions above are not met, the real-platform track will be omitted without weakening the core hypothesis test.

## Misuse risk and release strategy

A detector can support scam and forensic screening, but research artifacts can also reveal detector weaknesses or enable optimization against the detector. Therefore:

- release aggregate results and defensive methodology first;
- evaluate whether checkpoints or detailed attack recipes create avoidable misuse risk;
- never claim that one score proves an audio file is genuine or fake;
- document false-positive risks, especially across speaker groups and speaking styles;
- provide calibrated uncertainty or abstention where evidence is weak; and
- label the system as research software, not a replacement for human investigation.

## Fairness and limitations

Report performance by available demographic and speaking-condition groups only where consent, sample size, and privacy allow meaningful analysis. Avoid conclusions about age, region, accent, or disability from underpowered subgroups. A benchmark result does not establish reliability for all Thai speakers, all scams, or all future generators.

## Research integrity and AI-tool disclosure

YSC rules require the student to be responsible for the work and to disclose AI-tool use appropriately. Generative AI has been used to help review, structure, translate, and edit repository documentation. Before submission, the student researcher must:

1. independently verify every technical statement and citation;
2. rewrite or approve the text in their own words and be able to explain it;
3. keep a record of tools and the parts they assisted with;
4. include the disclosure and acknowledgement required by the current YSC rules; and
5. never use generated text or code as a substitute for conducting and understanding the experiment.

Suggested disclosure, to adapt to the official form:

> Generative-AI tools were used for language editing, translation assistance, document organization, and code-review support. The student researcher verified the content, selected the methodology, conducted the experiments, analyzed the results, and takes responsibility for the final work.

## Required review checklist

- [ ] Read the current YSC ethics statement and competition requirements
- [ ] Complete the student/adviser checklists and risk assessment
- [ ] Determine with the adviser/SRC whether new voice recording is human-participant research
- [ ] Obtain required SRC/IRB approval before experimentation
- [ ] Obtain informed consent/assent before recording or voice synthesis
- [ ] Record licenses and access approvals for every dataset and generator
- [ ] Approve a retention, access-control, and deletion plan
- [ ] Review real-platform experiments against current terms
- [ ] Add the final AI-tool disclosure to the YSC submission

## Official guidance

- YSC requirements: https://www.nstda.or.th/ysc/requirements/
- YSC forms and manuals: https://www.nstda.or.th/ysc/logo-and-forms-manuals/
- YSC website and ethics guidance: https://www.nstda.or.th/ysc/
