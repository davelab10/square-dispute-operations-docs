# Command Reference — v0.1.0

Exactly 16 commands. Provider-backed commands use the configured Station Vault credential and fixed Square environment host. Bounded reads expose completeness/continuation information where pagination applies.

## Discovery

### `square.dispute.list`
- **Purpose:** List disputes with optional state/location filters.
- **Effect:** Non-consequential read.
- **Inputs:** Optional `states`, `location_id`, `cursor`, `query_fingerprint`, `max_pages`, `max_items`.
- **Outputs:** Normalized dispute facts, derived data, and pagination/completeness information.
- **Provider:** Bounded Square dispute-list reads.
- **Governance:** No mutation; incomplete results remain distinguishable from complete scans.

### `square.dispute.actionable_queue`
- **Purpose:** Present a bounded queue of disputes surfaced for operator attention.
- **Effect:** Non-consequential read.
- **Inputs:** Optional `location_id`, `cursor`, `query_fingerprint`, `max_pages`, `max_items`.
- **Outputs:** Dispute facts, queue-oriented derived data, and completeness information.
- **Provider:** Bounded Square dispute-list reads.
- **Governance:** Organizes facts; does not decide or execute an accept/contest action.

### `square.dispute.deadline_queue`
- **Purpose:** Present disputes with deadline-oriented projections.
- **Effect:** Non-consequential read.
- **Inputs:** Optional `location_id`, `cursor`, `query_fingerprint`, `max_pages`, `max_items`, `horizon_hours`.
- **Outputs:** Dispute facts, deadline projections, and completeness information.
- **Provider:** Bounded Square dispute-list reads.
- **Governance:** A projection supports operator review and does not mutate a dispute.

### `square.dispute.inspect`
- **Purpose:** Inspect one dispute and its available payment projection.
- **Effect:** Non-consequential read.
- **Inputs:** Required `dispute_id`.
- **Outputs:** Normalized dispute and available related payment facts.
- **Provider:** Reads dispute and related payment projection when available.
- **Governance:** No mutation; unavailable related data is not treated as a case outcome.

## Case intelligence

### `square.dispute.evidence_requirements`
- **Purpose:** Show evidence suggestions associated with a dispute reason.
- **Effect:** Non-consequential local lookup.
- **Inputs:** Required `reason`.
- **Outputs:** Suggested evidence categories and bounded explanatory data.
- **Provider:** No Square request.
- **Governance:** Informational only; not legal advice or a finding of sufficiency.

## Evidence intelligence

### `square.dispute.evidence_inventory`
- **Purpose:** List evidence metadata for one dispute.
- **Effect:** Non-consequential read.
- **Inputs:** Required `dispute_id`; optional `cursor`, `query_fingerprint`, `max_pages`, `max_items`.
- **Outputs:** Evidence metadata and inventory completeness/continuation information.
- **Provider:** Bounded Square dispute/evidence reads.
- **Governance:** Metadata-oriented; does not fetch or validate evidence truth.

### `square.dispute.evidence_gap`
- **Purpose:** Compare observed evidence types with suggestions for the dispute reason.
- **Effect:** Non-consequential read plus local comparison.
- **Inputs:** Required `dispute_id`; optional `max_pages`, `max_items`.
- **Outputs:** Observed metadata, suggested categories, gap summary, and completeness.
- **Provider:** Reads dispute and bounded evidence inventory; compares results locally.
- **Governance:** An informational comparison, not a decision to contest.

### `square.dispute.evidence_text_prepare`
- **Purpose:** Validate text-evidence input locally and produce a commitment.
- **Effect:** Non-consequential local operation.
- **Inputs:** Required `dispute_id`, `evidence_type`, `evidence_text`.
- **Outputs:** Validation and commitment metadata; raw text is not echoed.
- **Provider:** No Square request.
- **Governance:** Does not create evidence or replace approval for the create command.

## Evidence mutation

### `square.dispute.evidence_text_create`
- **Purpose:** Create text evidence for a dispute.
- **Effect:** Consequential external mutation; native Airlock approval required.
- **Inputs:** Required `expected_environment`, `dispute_id`, `evidence_type`, `evidence_text`. Caller case/inventory fingerprints are not required.
- **Outputs:** Mutation/reconciliation status, text commitment, and bounded resulting evidence metadata; raw text is not echoed.
- **Provider:** Establishes internal case and complete-inventory baselines through bounded fresh reads, revalidates immediately before create, then verifies resulting metadata.
- **Governance:** Fails closed if case or inventory changed. Any bounded retry uses the same exact request; no universal exactly-once guarantee is made.

### `square.dispute.evidence_delete`
- **Purpose:** Delete one identified evidence item.
- **Effect:** Consequential external mutation; native Airlock approval required.
- **Inputs:** Required `expected_environment`, `dispute_id`, `evidence_id`, `expected_inventory_fingerprint`.
- **Outputs:** Mutation status and bounded post-operation inventory/reconciliation facts.
- **Provider:** Re-reads inventory, validates the expected binding, deletes the item, and observes state.
- **Governance:** Stale inventory is rejected; ambiguous deletion is not blindly retried.

## Accept governance

### `square.dispute.accept_preview`
- **Purpose:** Build a read-only plan to accept a dispute loss.
- **Effect:** Non-consequential provider read; proposed action is critical.
- **Inputs:** Required `dispute_id`.
- **Outputs:** Current case facts and a state-bound proposed acceptance plan.
- **Provider:** Reads current dispute/case state.
- **Governance:** Preview does not accept the dispute; acceptance is a concession.

### `square.dispute.accept_execute`
- **Purpose:** Execute the exact approved accept plan.
- **Effect:** Consequential external mutation; native Airlock approval required.
- **Inputs:** Required `plan` object from the preview path.
- **Outputs:** Execution status, observed mutation facts, and reconciliation reference.
- **Provider:** Revalidates relevant state before Square's accept operation and observes bounded outcome facts.
- **Governance:** Rejects invalid/stale plans. An ambiguous irreversible accept is not automatically retried.

## Contest governance

### `square.dispute.contest_preview`
- **Purpose:** Build a read-only plan to submit available dispute evidence.
- **Effect:** Non-consequential provider read; proposed action is high impact.
- **Inputs:** Required `dispute_id`; optional `max_pages`, `max_items` for evidence inventory.
- **Outputs:** Case/evidence facts, completeness, and a state-bound proposed submission plan.
- **Provider:** Reads dispute and bounded evidence metadata.
- **Governance:** Preview does not submit; incomplete inventory is not silently considered complete.

### `square.dispute.contest_execute`
- **Purpose:** Submit the exact approved contest plan.
- **Effect:** Consequential external mutation; native Airlock approval required.
- **Inputs:** Required `plan` object from the preview path.
- **Outputs:** Submission status, observed facts, and reconciliation reference.
- **Provider:** Revalidates relevant case/inventory state before submission and observes the response.
- **Governance:** Rejects invalid/stale plans. `PROCESSING` is not a win; ambiguous irreversible submissions are not automatically retried.

## Reconciliation

### `square.dispute.reconcile`
- **Purpose:** Re-read provider state related to a prior operation and classify the observation.
- **Effect:** Non-consequential provider read.
- **Inputs:** Required `reconciliation_ref` object from a supported operation.
- **Outputs:** Observed state, bounded classification, and uncertainty/limitations where applicable.
- **Provider:** Reads current Square dispute/evidence state referenced by the operation.
- **Governance:** Reconciles rather than retrying or claiming certainty beyond observed facts.

## Operations

### `square.dispute.operational_summary`
- **Purpose:** Produce a bounded summary of disputes and due-date workload.
- **Effect:** Non-consequential read plus local aggregation.
- **Inputs:** Optional `states`, `location_id`, `cursor`, `query_fingerprint`, `max_pages`, `max_items`, `due_within_hours`.
- **Outputs:** Aggregate counts/projections and completeness information.
- **Provider:** Bounded Square dispute-list reads.
- **Governance:** Operational context only; incomplete scans remain visible.
