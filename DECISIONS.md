# BullADM — architecture decisions

Short ADR-style notes for the public portfolio.

## ADR-001 — Telegram is an adapter, not a shell

**Decision:** the mobile/chat interface requests a bounded set of known operations instead of exposing arbitrary privileged commands.

**Why:** unrestricted command execution makes authorization, validation, retries and audit semantics too broad and dangerous.

**Trade-off:** fewer ad-hoc actions are available, but supported actions are explicit and testable.

## ADR-002 — prepare and confirm are separate phases

**Decision:** a mutating action is first inspected/preflighted and stored as a pending operation; execution requires a second explicit confirmation.

**Why:** the operator should see the exact planned scope before authorization, especially on a small mobile interface.

**Trade-off:** one extra interaction for privileged changes.

## ADR-003 — baseline validation is independent from artifact validity

**Decision:** a syntactically valid operation/artifact can still be rejected if the target baseline is unexpected.

**Why:** correctness of the change package and correctness of the current target state are different questions.

**Trade-off:** preflight requires extra state inspection before execution.

## ADR-004 — backup is a precondition, not a best effort

**Decision:** operations that require recoverability do not begin mutation until a checkpoint is created and considered valid.

**Why:** rollback cannot be a credible safety guarantee if the recovery material is discovered missing only after apply fails.

**Trade-off:** mutations may take longer and can fail before apply because recovery preparation failed.

## ADR-005 — verification is part of completion

**Decision:** a successful apply step does not immediately produce `completed`. Post-change verification must pass first.

**Why:** absence of an apply exception does not prove the application is healthy or the intended behaviour is present.

**Trade-off:** completion requires additional checks and explicit success criteria.

## ADR-006 — rollback is a normal workflow branch

**Decision:** failed verification transitions into rollback/recovery rather than treating rollback as an out-of-band emergency procedure.

**Why:** failure handling becomes testable, stateful and observable before an incident occurs.

**Trade-off:** the operation state machine is more complex.

## ADR-007 — duplicate delivery is expected

**Decision:** operation confirmation/status handling is designed to tolerate retries and duplicate client delivery.

**Why:** mobile/network clients can time out after the server has already processed a request.

**Trade-off:** operation identity and terminal-result storage are required for idempotent behaviour.

## ADR-008 — fail closed on operational ambiguity

**Decision:** missing/stale/ambiguous preconditions stop privileged execution.

**Why:** guessing is acceptable in some recommendation systems but not in a production mutation control path.

**Trade-off:** some operations require re-inspection/re-preflight even when the operator believes the change is safe.
