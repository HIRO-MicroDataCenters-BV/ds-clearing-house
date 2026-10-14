# ds-clearing-house

**The clearing house for Hiro dataspace services** — the per-node append-only
record of what happened across every service in the federation.

The clearing house is the audit substrate for the dataspace. Every service
that performs a meaningful transaction — a contract issuance, a credential
exchange, an access decision, a login, a federated-training run — writes an
immutable record here, keyed by a unique transaction identifier. The record
becomes the ground truth for regulatory review, dispute resolution, and
dataspace-wide traceability.

## Repository layout

| Directory | Purpose |
|-----------|---------|
| `server/` | FastAPI application + Helm chart + migrations |
| `server/app/` | Service code |
| `server/charts/server/` | Helm chart |
| `server/migrations/` | Alembic migrations |

## Service structure (initial)

- **Registry** — one row per issued contract; whether it is still `active`.
- **Ledger / audit history** — append-only log of what happened and when,
  keyed by a unique identifier per record.

### What this service deliberately does not (yet) do

- No hash chain or cryptographic witness over records — a database admin
  with write access can edit silently. Closing this gap needs signed
  checkpoints witnessed by peer nodes, which is a larger follow-up.
- Settlement, escrow, and the ethics body described in the NextGen proposal
  are not in scope here.

## Who writes to it

Every ds-* service that performs an auditable action — intended consumers:

| Service | What it writes |
|---|---|
| **ds-contract-engine** | Contract issued, accepted, revoked |
| **ds-auth** (the login service) | Session issuance (AID authenticated) |
| **ds-catalog** | Dataset created, updated, deleted |
| **ds-connector-service** | Exchange initiated, completed, failed |
| **ds-policy-engine** | Policy deployment events |
| **Cognitive Framework** | Federated training runs |

Every record carries a unique identifier so cross-service traces can be
reconstructed later.

## Reads

Reads are intended for:
- Dataspace operators auditing activity across the federation
- Regulators requesting the record of a specific transaction
- Dispute resolution between parties
- Observability dashboards (operator view, not raw record)

## Why this is its own repository

Previously this code lived in [`ds-contract-engine/server/clearing-house/`](
https://github.com/HIRO-MicroDataCenters-BV/ds-contract-engine) (added via
PR #5 by @rahulnl). Splitting it out matches the single-responsibility
principle used across the `ds-*` family and reflects the role the clearing
house plays in the architecture:

- It receives records from **many** services (not just the contract engine)
- Its retention + regulatory profile differs from any one operational service
- Its scaling profile is append-heavy, time-series-like — different from the
  catalog / policy engine shape
- The architecture diagram presents the clearing house as a peer of the
  contract engine, not a sub-component of it

See [SPLIT.md](./SPLIT.md) for the migration history.

## Development

```bash
cd server
cp .env.example .env   # if present; otherwise set env vars directly
poetry install
poetry run alembic upgrade head
poetry run uvicorn app.main:app --reload
```

## Deployment

Helm chart at `server/charts/server/`. Deploy via Fleet GitOps alongside the
rest of the `ds-*` services on each partner node.

## License

Apache-2.0
