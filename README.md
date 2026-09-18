# Square Dispute Operations

`dave/square-dispute-operations@0.1.0` is a 16-command Track 1 module for operating Square disputes through RailCall Station. It helps operators inspect a bounded dispute queue, understand evidence requirements and inventory, and carry out consequential actions through Station governance. It reports operational and provider facts; it does not make legal judgments or decide whether a dispute should be accepted or contested.

This repository documents the Track 1 Square capability toolbox. Track 2 is a separate orchestration/control-tower layer and is outside this module's scope.

## Capability map

| Purpose | Commands |
| --- | --- |
| Discovery | `square.dispute.list`, `square.dispute.actionable_queue`, `square.dispute.deadline_queue`, `square.dispute.inspect` |
| Case intelligence | `square.dispute.evidence_requirements` |
| Evidence intelligence | `square.dispute.evidence_inventory`, `square.dispute.evidence_gap`, `square.dispute.evidence_text_prepare` |
| Evidence mutation | `square.dispute.evidence_text_create`, `square.dispute.evidence_delete` |
| Accept governance | `square.dispute.accept_preview`, `square.dispute.accept_execute` |
| Contest governance | `square.dispute.contest_preview`, `square.dispute.contest_execute` |
| Reconciliation and operations | `square.dispute.reconcile`, `square.dispute.operational_summary` |

See [COMMANDS.md](COMMANDS.md) for the complete 16-command contract and inputs/outputs.

## Governed operating model

Consequential writes require native RailCall Station Airlock approval. Accept and Contest have separate preview and execute paths. Before consequential execution, the module revalidates relevant Square state and fails closed when that state is stale. `evidence_text_create` establishes its own bounded case and complete evidence-inventory baselines from fresh reads, then revalidates them before mutation; callers do not supply those fingerprints.

These protections reduce stale-action risk but are not an atomic compare-and-swap with Square. The module does not claim universal exactly-once behavior. When an irreversible operation has an ambiguous outcome, reconciliation observes provider state instead of blindly retrying it. `PROCESSING` is not proof that a dispute was won or lost.

## Install and configure

Install the existing module through the RailCall Marketplace:

```sh
railcall market install dave/square-dispute-operations
```

Configure the module's namespaced Square credential in Station Vault/Integrations with an access token and `environment` set to `sandbox` or `production`. Use Sandbox for evaluation. Keep credentials in Station's credential store; do not put them in source, command examples, receipts, or reports. Use the commands through Station when relying on native Airlock approvals and receipts.

## Quick start

1. Install the module and configure its Square credential in Station Vault/Integrations. Start with a Sandbox account when evaluating.
2. Use `square.dispute.list` to find a case, then `square.dispute.inspect` for its current facts.
3. Review requirements and evidence using `square.dispute.evidence_requirements`, `square.dispute.evidence_inventory`, and `square.dispute.evidence_gap`. `square.dispute.evidence_text_prepare` can validate text locally before any create request.
4. For an acceptance or contest, review the corresponding preview before submitting its execute command. Consequential mutations go through native Airlock approval.
5. If an irreversible operation has an uncertain outcome, use `square.dispute.reconcile` to observe provider state; do not assume it is safe to retry.

Choose only the steps relevant to the case. A preview is not an execution, and evidence suggestions are informational rather than legal advice.

## Verified evidence

The final known mocked/regression suite passed **92 tests**. Provider-backed direct-handler Square Sandbox E2E and native Station integration were established separately; neither is used to imply the other.

- **Direct-handler Sandbox E2E:** covered bounded reads and projections, evidence preparation/create/delete, Accept and Contest preview/execute, reconciliation, operational summary, and stale-state protections for Accept, Contest inventory, and evidence creation.
- **Native Station integration:** 16/16 commands were discovered; Sandbox Vault resolution and representative reads/local preparation worked; an Airlock-governed `evidence_text_create` succeeded with exactly one observed Sandbox evidence creation. Native receipt signature verification passed **6 checks**. Raw evidence text and the token were absent from command output and the receipt.

The direct-handler Sandbox results are not Station/Airlock evidence. See [TESTED.md](TESTED.md) for the established details and boundaries.

## Limitations

- v0.1.0 supports text evidence only; file and attachment upload are not supported.
- Evidence inventory is metadata-oriented; it does not fetch or establish the truth of evidence content.
- `PROCESSING` is not a `WON` or `LOST` outcome.
- Ambiguous irreversible Accept and Submit operations are not automatically retried. No universal exactly-once guarantee is claimed.

## Evidence and documentation

- [COMMANDS.md](COMMANDS.md) — complete buyer-facing reference for all 16 commands.
- [TESTED.md](TESTED.md) — established regression, direct-handler Square Sandbox E2E, and separate native Station evidence.
