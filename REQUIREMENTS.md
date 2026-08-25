# BullADM — requirements specification

This document describes the public-safe behavioural requirements of the BullADM operational workflow.

## Actors

- **Operator** — reviews and authorizes known operational actions.
- **Admin interface** — mobile/chat adapter that presents status and confirmation.
- **Control gateway** — authenticates and validates requests.
- **Operation service** — owns workflow state and safety rules.
- **Application target** — state affected by an approved operation.

## Functional requirements

### FR-01 — Inspect operation input
The system shall inspect an operation artifact/request before it can become executable.

**Acceptance criteria**
- malformed/unsupported input is rejected;
- inspection has no production side effect;
- inspection result can be shown to the operator.

### FR-02 — Validate expected baseline
The system shall validate that the target state matches the expected baseline before authorization/execution.

**Acceptance criteria**
- baseline mismatch blocks execution;
- ambiguous baseline is treated as failure;
- the operator is not offered a misleading success path.

### FR-03 — Create pending operation
A valid inspected request shall become a stored pending operation with an immutable operation identity.

**Acceptance criteria**
- pending state contains the planned action summary;
- creating it does not apply the change;
- a new materially different action requires a new operation identity.

### FR-04 — Require explicit confirmation
A state-changing operation shall require explicit operator confirmation.

**Acceptance criteria**
- no confirmation means no mutation;
- expired confirmation cannot execute;
- confirmation applies the stored operation, not arbitrary replacement parameters.

### FR-05 — Create recovery checkpoint
The system shall create a valid recovery checkpoint before mutation begins.

**Acceptance criteria**
- backup/checkpoint failure blocks apply;
- the operation records that a recovery path exists;
- apply does not start while backup state is ambiguous.

### FR-06 — Apply only approved scope
The system shall apply only the scope described by the confirmed operation.

**Acceptance criteria**
- the target scope is known before apply;
- unrelated state is not intentionally modified;
- unsupported generic shell execution is outside the public workflow.

### FR-07 — Verify after apply
The system shall run post-change verification after mutation.

**Acceptance criteria**
- success is not reported before verification;
- failed verification enters recovery handling;
- verification result is part of the operation status.

### FR-08 — Roll back failed changes
The system shall attempt rollback when post-apply verification fails and a valid checkpoint exists.

**Acceptance criteria**
- rollback restores from the operation's checkpoint;
- recovery is verified after restore;
- rollback success and rollback failure are distinct outcomes.

### FR-09 — Support idempotent retries
Repeated delivery of the same confirmation/result request shall not repeat a completed business effect.

**Acceptance criteria**
- an already completed operation is not applied again;
- an already rolled-back operation is not re-applied by a duplicate confirmation;
- clients may receive the stored terminal result.

### FR-10 — Expose operation status
The operator shall be able to understand the current operation state from the mobile interface.

**Acceptance criteria**
- pending, applying, verifying, completed, rolled-back and failed states are distinguishable;
- errors identify the workflow stage at which execution stopped;
- status text does not require access to a server shell to interpret.

## Non-functional requirements

### NFR-01 — Fail closed
Unknown, stale or ambiguous privileged state must stop the operation rather than guess.

### NFR-02 — Least authority
The operator interface should expose a narrow set of known actions instead of unrestricted privileged command execution.

### NFR-03 — Auditability
Important state transitions and terminal outcomes should be reviewable after the operation.

### NFR-04 — Recoverability
Recovery must be designed before mutation, not improvised after failure.

### NFR-05 — Mobile readability
An operator should be able to review the important action, confirmation and outcome information on a phone.

### NFR-06 — Separation of concerns
UI adapters must not own the safety rules for preflight, confirmation, backup, apply or rollback.

### NFR-07 — Secret minimization
Operational status and previews must not require exposing credentials or cryptographic secrets to the chat interface.

## Out of scope for the public repository

- real authentication material;
- production endpoints/hostnames/IPs;
- privileged shell commands;
- provider/account identifiers;
- backup contents or production logs;
- implementation code that would expose administrative access.
