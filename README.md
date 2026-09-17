# Square Dispute Operations

`dave/square-dispute-operations` is a RailCall module for bounded Square dispute discovery, evidence metadata/intelligence, and operator-governed dispute actions. It reports provider facts and bounded operational summaries; it does not make legal judgments, decide whether a case should be accepted or contested, or create/charge payments.

## Capabilities

The 16 commands are:

| Area | Commands |
| --- | --- |
| Discovery | `square.dispute.list`, `square.dispute.actionable_queue`, `square.dispute.deadline_queue`, `square.dispute.inspect` |
| Evidence intelligence | `square.dispute.evidence_inventory`, `square.dispute.evidence_requirements`, `square.dispute.evidence_gap`, `square.dispute.evidence_text_prepare` |
| Governed evidence actions | `square.dispute.evidence_text_create`, `square.dispute.evidence_delete` |
| Accept / contest | `square.dispute.accept_preview`, `square.dispute.accept_execute`, `square.dispute.contest_preview`, `square.dispute.contest_execute` |
| Follow-up | `square.dispute.reconcile`, `square.dispute.operational_summary` |

Discovery reads are bounded and expose completeness/continuation information instead of silently treating a capped result as exhaustive. The queues and evidence-gap command organize observed facts; operators remain responsible for case decisions and supporting documentation.

## Governance and freshness

The write commands are declared `write_requires_approval` and should be run through RailCall Station's native Airlock. A staged request is a proposal, not an execution. Station approval is bound to the exact command input; changing the payload requires a new approval.

Accept and contest use a previewed plan and an execute command that receives that exact plan. Before an irreversible operation, the module re-reads relevant Square state and rejects stale case, environment, or evidence-inventory bindings. These checks reduce stale-action risk but are not an atomic compare-and-swap with Square. An accepted dispute is a concession; a submitted contest is not a promise that the dispute will be won.

Evidence text creation is text-only in v0.1.0. `evidence_text_prepare` locally validates the exact text and returns a commitment without echoing the text. The create command binds the expected environment, case fingerprint, and inventory fingerprint, then verifies the provider's resulting evidence metadata. Evidence deletion is also a governed write. This module does not upload files or attachments.

Irreversible accept/contest operations and evidence deletion are not blindly retried after ambiguous outcomes; reconcile before deciding what to do next. Evidence text creation has only a bounded retry using the same exact request and idempotency identity. It never retries with altered content.

## Installation and credentials

Install the module using the normal RailCall Marketplace path:

```sh
railcall market install dave/square-dispute-operations
```

Configure the module's namespaced Square credential in Station Integrations/Vault. Required fields are:

- `access_token`: a Square access token, stored only in the Station credential store;
- `environment`: `sandbox` or `production`.

The environment selects Square's Sandbox or production API host. Use a Sandbox token with `environment=sandbox` for test accounts. Never place a real token in source, examples, command arguments, receipts, or public documentation.

Invoke commands through Station. For example, a read uses `square.dispute.list`; inspect requires a dispute ID. Accept and contest require operator review of the previewed plan and native approval before execution. Evidence creation likewise requires an approved exact payload. Do not call the handler directly when claiming Station Airlock governance or Station receipts.

## Bounded reads and privacy

List, queue, inventory, and summary operations accept bounded pagination controls where applicable. Outputs report completeness and continuation state so callers can distinguish a complete scan from a partial one. The module uses fixed Square hosts and does not accept arbitrary request URLs.

Module responses and normalized provider errors avoid echoing the access token and raw evidence text. The exact evidence text is necessarily part of the proposed write for operator review and is sent to Square if approved. Apply the local Station's receipt/log retention policy accordingly. Evidence inventory is metadata-oriented; the module does not claim to validate the truth of submitted evidence.

## Sandbox compatibility and limitations

Square Sandbox responses can differ from production. The bounded compatibility cases covered by v0.1.0 include Sandbox disputes whose object `version` is absent or null and Sandbox inventories that omit an empty evidence array. The module tolerates only the explicitly supported cases; production validation remains stricter. A preview can become stale between its final preflight read and the provider write because Square does not provide an atomic compare-and-swap for this operation.

This module does not create payments, accept or contest automatically, assess evidence truth, provide legal advice, upload attachments, or guarantee a final dispute outcome. Reconciliation reports observed state and bounded causal limits; it does not turn an ambiguous provider response into certainty.
