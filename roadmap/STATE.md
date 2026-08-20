# DASP implementation state

This file is the ground-truth implementation queue. Update it only when repository reality changes. A scaffold is not completion.

| ID | Status | Deliverable | Depends on |
|---|---|---|---|
| PROTO-00 | ✅ done | Repository bootstrap, Apache-2.0, canonical spec, agent rules | — |
| NARR-DRAFT | ✅ done | Non-binding provider-neutral Narrative Runtime Profile vocabulary draft for cross-provider alignment | PROTO-00 |
| PROTO-01 | ⬜ pending | Verify/finalize entity IDs + envelope schemas and RED/green fixtures | PROTO-00 |
| PROTO-02 | ⬜ pending | Node lifecycle + capability discovery schemas/fixtures | PROTO-01 |
| PROTO-03 | ⬜ pending | Event + Intent payload schemas and catalogs; explicit participant attribution conventions | PROTO-01 |
| PROTO-04 | ⬜ pending | Action request/cancel/result schemas; idempotency + TTL fixtures | PROTO-01, PROTO-02 |
| PROTO-05 | ⬜ pending | Transform/frame/unit schemas + stream descriptor | PROTO-01 |
| PROTO-06 | ⬜ pending | 20+ language-neutral valid/invalid conformance vectors | PROTO-02, PROTO-03, PROTO-04, PROTO-05 |
| PROTO-07 | ⬜ pending | Python validator/reference client + fixture runner | PROTO-06 |
| PROTO-08 | ⬜ pending | TypeScript validator/client + same fixtures | PROTO-06 |
| PROTO-09 | ⬜ pending | C++/Unreal-friendly model/validator strategy + same fixtures | PROTO-06 |
| PROTO-10 | ⬜ pending | Tag `v0.1.0-alpha.1`; compatibility/versioning notes | PROTO-07, PROTO-08, PROTO-09 |
| NARR-00 | ⬜ pending | Ratify provider-neutral Narrative Runtime Profile design against at least two provider/runtime perspectives | PROTO-10, NARR-DRAFT |
| NARR-01 | ⬜ pending | Persona/utterance/addressing/perception/quest-state profile schemas + fixtures | NARR-00 |

## Immediate next row

`PROTO-01`

Starter schemas exist, but they have **not** passed the full conformance and cross-language review required by PROTO-01/04. Treat them as implementation inputs, not already-ratified completion.

The Narrative Runtime Profile draft is intentionally non-normative until `NARR-00`/`NARR-01`; it exists now to align provider vocabulary without blocking core alpha work.
