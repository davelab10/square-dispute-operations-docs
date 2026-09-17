# Square Dispute Operations

`dave/square-dispute-operations` v0.1.0 is a Track 1 RailCall module for bounded Square dispute operations. It gives operators a structured view of disputes and evidence metadata, then routes consequential actions through native RailCall Station governance. It reports observed facts; it does not make legal judgments or decide whether a dispute should be accepted or contested.

## What it covers

Exactly 16 commands span:

| Purpose | Commands |
| --- | --- |
| Discovery | `square.dispute.list`, `square.dispute.actionable_queue`, `square.dispute.deadline_queue`, `square.dispute.inspect` |
| Case intelligence | `square.dispute.evidence_requirements` |
| Evidence intelligence | `square.dispute.evidence_inventory`, `square.dispute.evidence_gap`, `square.dispute.evidence_text_prepare` |
| Evidence mutation | `square.dispute.evidence_text_create`, `square.dispute.evidence_delete` |
| Accept governance | `square.dispute.accept_preview`, `square.dispute.accept_execute` |
| Contest governance | `square.dispute.contest_preview`, `square.dispute.contest_execute` |
| Reconciliation and operations | `square.dispute.reconcile`, `square.dispute.operational_summary` |

See [COMMANDS.md](COMMANDS.md) for the complete command reference.

## Governed operations

Consequential writes require native RailCall Station Airlock approval. Accept and contest are separate preview and execute commands; execution is tied to the approved plan and revalidates relevant Square state. Evidence create establishes case and complete-inventory baselines internally from bounded fresh reads, then revalidates before mutation. The caller does not provide case or inventory fingerprints. Stale state fails closed. After an operation, reconciliation observes provider state instead of blindly retrying an ambiguous irreversible action.

These checks reduce stale-action risk but are not an atomic compare-and-swap with Square. The module does not claim universal exactly-once behavior or guarantee an outcome from a submitted contest.

## Verified evidence

The established evidence includes a final mocked/regression suite of **92 passing tests**, direct-handler Square Sandbox E2E coverage, and a separate native Station integration. The Station evidence includes 16/16 command discovery, Sandbox Vault resolution, a successful Airlock-governed evidence create, one observed Sandbox creation, and receipt signature verification passing six checks. Details and boundaries are in [TESTED.md](TESTED.md).

## Installation and configuration

Install through the RailCall Marketplace:

```sh
railcall market install dave/square-dispute-operations
```

Configure the module's namespaced Square credential in Station Vault/Integrations with an access token and `environment` set to `sandbox` or `production`. Use a Sandbox credential for test accounts. Keep credentials in Station's credential store; never put them in source, command examples, receipts, or reports. Use commands through Station when relying on Airlock approvals and native receipts.

## Limitations

Evidence submission is text-only in v0.1.0; file and attachment upload are not supported. A Sandbox contest observed in `PROCESSING` is not a win or loss. Evidence inventory is metadata-oriented and does not verify evidence truth. Irreversible Accept and Submit operations are not automatically retried when their outcome is ambiguous. See [ARCHITECTURE.md](ARCHITECTURE.md) and [SECURITY.md](SECURITY.md) for the operating and safety boundaries.
