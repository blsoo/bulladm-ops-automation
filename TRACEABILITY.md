# BullADM — requirements traceability

This matrix connects analyst requirements to workflow behaviour and verification evidence.

| Requirement | Workflow / rule | Contract evidence | Verification |
|---|---|---|---|
| FR-01 Inspect operation input | BR-04 | Inspect section | TS-01, TS-02 |
| FR-02 Validate expected baseline | BR-05, BR-06 | `BASELINE_MISMATCH` | TS-03, TS-04 |
| FR-03 Create pending operation | BR-01..BR-03 | operation resource / prepare result | TS-03, TS-14 |
| FR-04 Require explicit confirmation | BR-07..BR-10 | Confirm section | TS-05, TS-06, TS-14 |
| FR-05 Create recovery checkpoint | BR-11, BR-12 | `BACKUP_FAILED` | TS-07 |
| FR-06 Apply only approved scope | BR-02, BR-08, BR-19 | narrow operation contract | TS-14, TS-15 |
| FR-07 Verify after apply | BR-13 | status / terminal success | TS-08, TS-09 |
| FR-08 Roll back failed changes | BR-14, BR-15 | terminal rollback | TS-09, TS-10, TS-11 |
| FR-09 Support idempotent retries | BR-10, BR-16, BR-17 | idempotency table | TS-12, TS-13 |
| FR-10 Expose operation status | BR-21, BR-22 | status response / error stage | TS-07..TS-13 |
| NFR-01 Fail closed | BR-05, BR-06, BR-09, BR-12 | error model | TS-02, TS-04, TS-06, TS-07 |
| NFR-02 Least authority | BR-19 | narrow contract | TS-15 |
| NFR-03 Auditability | state machine + terminal states | stable operation id/result | TS-08..TS-13 |
| NFR-04 Recoverability | BR-11..BR-15 | rollback contract | TS-07, TS-09..TS-11 |
| NFR-05 Mobile readability | BR-21, BR-22 | status/preview shape | system UX review |
| NFR-06 Separation of concerns | BR-18 | control-plane diagrams | architecture review |
| NFR-07 Secret minimization | BR-20 | public contract excludes secrets | repository safety review |

## Change-impact example

If confirmation expiry semantics change, review at minimum:

- FR-04;
- BR-09;
- operation state machine;
- `CONFIRMATION_EXPIRED` contract behaviour;
- TS-06;
- duplicate/retry semantics where a delayed client may resend an old request.

The goal of traceability is to make that impact visible without reading implementation code first.
