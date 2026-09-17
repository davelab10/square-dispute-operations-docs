# Security and Safety Model

- **Credentials:** Square credentials are configured in the module's namespaced Station Vault/Integrations entry. They are not included in this repository or command outputs.
- **Provider destinations:** Requests use the fixed Square Sandbox or production host selected by the configured environment; arbitrary URLs are not accepted.
- **Least capability:** The module has no `PAYMENTS_WRITE` capability and no payment-creation command.
- **Privacy:** Outputs avoid returning the access token and do not echo raw evidence text. Evidence inventory is metadata-oriented. The exact text is supplied for an approved create and sent to Square only as part of that operation.
- **Approval:** Consequential evidence and dispute actions use native Station Airlock approval. Preview and execution are separate for accept/contest; changing an approved request requires a new approval.
- **Freshness:** Relevant case and inventory state is re-read before mutation. A stale binding fails closed; the read-then-write sequence is not atomic with Square.
- **Bounded reads:** Pagination is bounded and completeness/continuation is surfaced so a capped result is not mistaken for a complete scan.
- **Ambiguous outcomes:** Reconciliation observes provider state; irreversible Accept and Submit operations are not automatically retried after ambiguous results.
- **Content boundary:** v0.1.0 supports text evidence only. It has no file or artifact upload path.

These controls reduce operational risk; they do not determine the truth or legal sufficiency of evidence and do not guarantee dispute outcomes.
