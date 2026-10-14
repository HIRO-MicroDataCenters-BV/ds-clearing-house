# Migration note: split from ds-contract-engine

This repository contains the code originally merged by @rahulnl into
[`ds-contract-engine` PR #5](https://github.com/HIRO-MicroDataCenters-BV/ds-contract-engine/pull/5)
at path `server/clearing-house/`. All attribution for the service logic
belongs to that PR.

## Why we split

`ds-contract-engine` is responsible for contract issuance, acceptance and
revocation. The clearing house is responsible for recording what happened
across the federation — contracts, exchanges, access decisions, login
sessions, federated training runs, policy deployments. Many services write
to it; only the contract engine writes to its own contract tables.

Keeping them in one repo would have conflated their lifecycles and made it
harder to let other `ds-*` services write to the clearing house without
also depending on the contract engine.

Each repository now stays focused on its single responsibility, matching the
convention used by `ds-catalog`, `ds-policy-engine`, `ds-connector-service`
and `ds-search-service`.

## Follow-up required in ds-contract-engine

- Remove `server/clearing-house/` from `develop` (keep `clearing-house-stub`
  for local compose if useful).
- Reference `ds-clearing-house` as an external service in docker-compose
  and in the production deployment.
- Any imports from `app.core.models.contract` etc. that used to live in the
  same tree will need to call the clearing-house over HTTP instead.

A tracking issue has been opened on `ds-contract-engine` to coordinate the
removal.
