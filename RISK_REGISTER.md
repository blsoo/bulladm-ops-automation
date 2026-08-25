# BullADM risk register

Public-safe risk analysis for the operational workflow. Real infrastructure and credentials are intentionally excluded.

| ID | Risk | Likelihood | Impact | Control / mitigation | Verification |
|---|---|---:|---:|---|---|
| R-01 | Operator confirms the wrong action | Medium | High | Exact preview + operation ID + expiry | confirmation tests |
| R-02 | Valid change is applied to unexpected baseline | Medium | High | preflight baseline validation; fail closed | baseline mismatch scenario |
| R-03 | Duplicate confirmation repeats mutation | Medium | High | idempotent operation state | duplicate-confirmation test |
| R-04 | Backup fails but apply continues | Low | Critical | backup success is hard precondition | backup-failure test |
| R-05 | Apply succeeds partially | Low/Medium | High | narrow change scope + post-apply verification + rollback | partial/failure scenario |
| R-06 | Verification is too weak and reports false success | Medium | High | define explicit health/acceptance checks | verification test matrix |
| R-07 | Rollback runs but recovery is not verified | Low | Critical | rollback has its own verification stage | recovery verification test |
| R-08 | Stale confirmation authorizes outdated operation | Medium | High | expiry + re-validation before execution | stale-operation test |
| R-09 | Telegram/control UI becomes generic shell access | Low | Critical | allowlisted operation types and narrow control contract | security boundary review |
| R-10 | Sensitive infrastructure leaks into public artifacts | Low | High | public-safe content policy + automated pattern checks | CI portfolio checks |

## Risk treatment principles

1. **Prevent** ambiguous actions before execution.
2. **Detect** state mismatch through preflight/verification.
3. **Recover** with a tested rollback path.
4. **Prove** the outcome through explicit terminal states and audit/status evidence.

## Why keep a risk register

The same workflow diagram can look safe while hiding unhandled failure modes. A risk register forces each important failure to have an owner in the design: a requirement, business rule, test or operational control.
