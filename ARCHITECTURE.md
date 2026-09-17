# Architecture

Square Dispute Operations separates observation, operator intent, governed execution, and post-action observation:

```text
intent / bounded read
        ↓
preview and state-bound plan (where applicable)
        ↓
native RailCall Station Airlock approval
        ↓
fresh Square state validation
        ↓
approved mutation
        ↓
authoritative reread and reconciliation
```

Read and queue commands return bounded provider facts and make pagination completeness visible. Evidence inventory is metadata-oriented. Local evidence preparation validates text and produces a commitment without creating a provider record.

For accept and contest, preview produces the proposal and execution consumes the approved plan. Station's native approval binds the exact request. Before mutation, the module re-reads relevant dispute and evidence state and rejects stale bindings. This reduces stale-action risk but is not an atomic compare-and-swap with Square.

For text evidence creation, callers provide the expected environment, dispute ID, evidence type, and exact text. The module establishes case and complete-inventory fingerprints internally from bounded reads and revalidates them immediately before create. The create request uses a deterministic idempotency identity; that does not support a universal exactly-once claim. Irreversible ambiguous operations are handled through reconciliation, not blind retries.

Reconciliation re-reads available provider state and reports what can be observed and what remains uncertain. It does not claim that evidence is truthful or that a submitted contest will win.
