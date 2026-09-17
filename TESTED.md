# Verification Evidence — v0.1.0

Only already-established evidence is listed here.

## Unit / regression

- Final known suite after the `evidence_text_create` contract change: **92 passing tests**.
- This is mocked/unit regression evidence, not live-provider or Station evidence.

## Direct-handler Square Sandbox E2E

Provider-backed direct-handler coverage included:

- dispute discovery, actionable/deadline reads, inspect, and payment projection;
- evidence inventory, requirements, gap, and text preparation;
- text evidence create/delete;
- Accept preview/execute;
- Contest preview/execute, with the Sandbox submission observed in `PROCESSING`;
- reconciliation and operational summary;
- stale Accept rejected with zero module Accept mutation;
- stale Contest inventory rejected with zero `SubmitEvidence` mutation;
- stale evidence-create rejected with zero create mutation.

This direct-handler E2E is distinct from native Station integration and does not establish Airlock or Station receipt behavior on its own.

## Native Station integration

Separately established through Station:

- signed module loaded; 16/16 commands discovered;
- Sandbox credential resolved through Station Vault;
- representative reads and local text preparation succeeded;
- native Airlock-governed `evidence_text_create` succeeded;
- exactly one Sandbox evidence creation was observed;
- receipt signature verification passed with **6 checks**;
- raw evidence text and token were absent from command output and receipt.

## Limitations and interpretation

- v0.1.0 supports text evidence only; it does not upload files or attachments.
- Sandbox `PROCESSING` is not a `WON` or `LOST` outcome.
- No universal exactly-once guarantee is claimed. Ambiguous irreversible Accept and Submit operations are not automatically retried.
- Direct-handler Sandbox results and native Station evidence are separate test paths.
- Known bounded Sandbox compatibility observations: a dispute object's `version` may be absent/null, and an empty evidence array may be omitted. These observations do not imply identical production responses.
- No additional test or provider result is implied.
