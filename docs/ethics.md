# Ethics, Consent, and Responsible Use Plan

## Status

This is a pre-study governance plan, not ethics approval. The student researcher and adviser must use the current YSC forms and obtain every required SRC, IRB, or institutional approval before protected-data use or participant recording.

BenchmarkGap-TH is designed to use existing, licensed speech datasets and deterministic offline G.711 transformations. It does not require placing telephone calls, using messaging platforms, or exposing audio to third-party transmission services.

## Existing datasets

For every dataset, record:

- source and version;
- license and permitted research purpose;
- access approval and expiration, if any;
- redistribution restrictions;
- speaker and consent documentation supplied by the dataset owner; and
- required deletion or retention conditions.

Restricted audio must not be committed to GitHub. Technical accessibility does not create permission to download, redistribute, clone, or publish a person's voice.

## New recordings, if required

New participant recording is outside the preferred scope. If a Thai dataset cannot be obtained and the team considers collecting speech, the protocol must be treated as a new project activity requiring adviser and YSC/SRC/IRB review before recording begins.

Consent material must explain:

- what is recorded and why;
- whether speech may be transformed or synthetically generated;
- who can access raw and derived audio;
- retention, deletion, withdrawal, and publication rules;
- foreseeable privacy and voice-impersonation risks; and
- researcher, adviser, and reviewing-body contact details.

Minors require the assent and guardian-consent process specified by the responsible review body. No public-figure, teacher, classmate, or other non-consenting voice may be cloned for this project.

## Data minimization and security

- Use random source and speaker identifiers in analysis files.
- Keep identity keys and consent records separate and encrypted.
- Collect only metadata required for splitting and analysis.
- Store restricted audio in access-controlled storage, not the repository.
- Treat speaker embeddings as potentially identifying biometric data.
- Define retention and deletion dates before data use.
- Publish aggregate results and approved, de-identified artifacts only.
- Do not commit credentials, dataset tokens, identities, or restricted manifests.

The public repository may contain code, configuration, schemas, hashes, aggregate tables, and artifacts whose licenses explicitly permit release.

## G.711 transformation safety

C0–C3 are offline transformations of the same approved source recordings. No audio will be sent through an actual phone network. The transformed copies inherit the source dataset's access and redistribution restrictions and must be deleted on the same schedule unless the approval states otherwise.

Every condition must be applied symmetrically to genuine and spoof audio. Asymmetric processing could create a misleading classifier shortcut and an invalid result.

## Responsible interpretation

- A low EER does not prove that an individual recording is genuine.
- A measured G.711 gap does not establish performance on real Thai phone networks.
- Report false-positive and false-negative consequences, not accuracy alone.
- Avoid claims about demographic groups with insufficient sample size.
- Do not market the detector as a forensic, banking, or law-enforcement decision system.
- Preserve negative and inconclusive results; do not remove difficult conditions after scoring.

## Misuse and release

The study primarily measures detector fragility. Detailed adversarial optimization recipes are outside scope. Before releasing checkpoints or source-level scores, assess whether they expose restricted voices, enable impersonation, or facilitate evasion. Defensive analysis and aggregate results should be released before higher-risk artifacts.

## Research integrity and AI-tool disclosure

YSC rules require the student to remain responsible for the work and disclose AI-tool use appropriately. Generative AI has assisted with reviewing, structuring, translating, and editing repository documentation. Before submission, the student researcher must:

1. independently verify every claim, equation, number, and citation;
2. approve the wording and be able to explain the full methodology;
3. keep a record of tools and the parts they assisted with;
4. include the disclosure and acknowledgement required by current YSC rules; and
5. conduct and understand the experiment rather than treating generated text or code as evidence.

Suggested disclosure to adapt to the official form:

> Generative-AI tools were used for language editing, translation assistance, document organization, and code-review support. The student researcher verified the content, selected the methodology, conducted the experiments, analyzed the results, and takes responsibility for the final work.

## Required checklist

- [ ] Read current YSC ethics and competition requirements
- [ ] Complete student/adviser checklists and risk assessment
- [ ] Obtain the required SRC/IRB decision before protected-data use
- [ ] Record the license and approval for every dataset
- [ ] Approve access-control, retention, and deletion procedures
- [ ] Confirm that C0–C3 transformations remain within permitted use
- [ ] Audit the public repository for restricted data and identifiers
- [ ] Add the final AI-tool disclosure to the submission

## Official guidance

- YSC requirements: https://www.nstda.or.th/ysc/requirements/
- YSC forms and manuals: https://www.nstda.or.th/ysc/logo-and-forms-manuals/
- YSC website and ethics guidance: https://www.nstda.or.th/ysc/
