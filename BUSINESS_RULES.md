# BullADM — operational business rules

These rules make the privileged workflow explicit instead of hiding safety behaviour in UI handlers.

## Identity and scope

**BR-01 — Stable operation identity**  
Every prepared operation has a stable identifier. Confirmation references that identity rather than reconstructing the action from free-form text.

**BR-02 — Scope is fixed before confirmation**  
The planned target/scope must be known before the operator authorizes execution.

**BR-03 — Material change means new operation**  
If target, action type or meaningful parameters change, the system creates a new pending operation.

## Inspection and preflight

**BR-04 — Inspection has no mutation side effect**  
Reading or validating an artifact/request cannot itself alter application state.

**BR-05 — Baseline mismatch blocks execution**  
A valid artifact is not automatically safe for an unexpected target baseline.

**BR-06 — Ambiguity is failure**  
If the service cannot establish the required precondition, it stops instead of selecting a likely state.

## Confirmation

**BR-07 — Intent is not authorization**  
Recognizing an operator request does not authorize the mutation.

**BR-08 — Confirmation applies stored scope**  
Confirmation authorizes the already prepared operation. It cannot silently replace the target or operation payload.

**BR-09 — Confirmation expires**  
A stale pending operation must be inspected/preflighted again before it can execute.

**BR-10 — Terminal operations cannot be revived by duplicate confirmation**  
A completed, rolled-back, rejected or expired operation is not restarted by resending the same confirmation.

## Recovery

**BR-11 — Backup before apply**  
A valid recovery checkpoint is a hard precondition for a mutating operation that requires rollback capability.

**BR-12 — Failed backup means no apply**  
If checkpoint creation cannot be verified, the mutation does not start.

**BR-13 — Verification precedes success**  
An operation is not `completed` merely because the apply step returned without an immediate error.

**BR-14 — Failed verification enters recovery**  
When post-apply checks fail, recovery is the normal next workflow branch.

**BR-15 — Recovery itself must be verified**  
Rollback success is not assumed; restored state is checked before the operation is marked `rolled_back`.

## Idempotency and retries

**BR-16 — One operation, one business effect**  
Retries may repeat delivery but must not repeat the intended production mutation.

**BR-17 — Stored terminal result is reusable**  
A retry after timeout may receive the existing terminal result instead of executing again.

## Interface and authority

**BR-18 — Admin UI is an adapter**  
Telegram/mobile UI does not own preflight, confirmation, backup, apply or rollback business rules.

**BR-19 — No generic remote shell**  
The safe workflow exposes known operations with bounded authority rather than arbitrary privileged command execution.

**BR-20 — Secrets are not UI data**  
Credentials and cryptographic secrets are not included in operator previews or routine status output.

## Status semantics

**BR-21 — State names have one meaning**  
`pending_confirmation`, `applying`, `verifying`, `completed`, `rollback`, `rolled_back`, `rejected`, `expired` and `failed` represent distinct workflow conditions.

**BR-22 — Failure stage is observable**  
The operator should be able to distinguish inspection failure, preflight failure, backup failure, apply failure, verification failure and recovery failure.
