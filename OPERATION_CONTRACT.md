# BullADM — public operation contract

This is a public-safe behavioural contract. It documents the shape of the workflow without exposing real endpoints, authentication material or production infrastructure.

## Operation resource

```json
{
  "operation_id": "op_7f3a",
  "type": "apply_artifact",
  "state": "pending_confirmation",
  "summary": "Apply reviewed change set to the expected application baseline",
  "baseline": {
    "status": "matched"
  },
  "recovery": {
    "required": true,
    "checkpoint_state": "not_created"
  },
  "expires_at": "2026-08-26T10:15:00Z"
}
```

The example values are synthetic.

## Conceptual operations

### Inspect

Input:

```json
{
  "operation_type": "apply_artifact",
  "artifact_ref": "artifact_example_001"
}
```

Output:

```json
{
  "inspection": "passed",
  "supported": true,
  "planned_scope": ["component-a"],
  "requires_confirmation": true
}
```

Properties:

- read/validation only;
- no target mutation;
- unsupported input is rejected before a pending operation is created.

### Prepare / preflight

Conceptual result:

```json
{
  "operation_id": "op_7f3a",
  "state": "pending_confirmation",
  "baseline": "matched",
  "preview": "Apply reviewed change set to component-a",
  "expires_at": "2026-08-26T10:15:00Z"
}
```

The preview is derived from the stored operation and must be enough for the operator to understand the intended scope.

### Confirm

Conceptual request:

```json
{
  "operation_id": "op_7f3a",
  "decision": "confirm"
}
```

The confirmation request does **not** include a replacement target or arbitrary command string. It authorizes the stored pending operation.

### Status

Conceptual response:

```json
{
  "operation_id": "op_7f3a",
  "state": "verifying",
  "stage": "post_apply_checks",
  "message": "Change applied; verification in progress"
}
```

### Terminal success

```json
{
  "operation_id": "op_7f3a",
  "state": "completed",
  "applied": true,
  "verification": "passed"
}
```

### Terminal rollback

```json
{
  "operation_id": "op_7f3a",
  "state": "rolled_back",
  "applied": false,
  "verification": "failed",
  "recovery_verification": "passed"
}
```

## State model

Allowed public conceptual states:

```text
received
inspected
rejected
preflight_passed
pending_confirmation
expired
backup_creating
applying
verifying
completed
rollback
rolled_back
failed
```

Not every state is client-writable. Most transitions are controlled by the operation service.

## Error model

```json
{
  "error": {
    "code": "BASELINE_MISMATCH",
    "stage": "preflight",
    "message": "Target state does not match the expected baseline"
  }
}
```

Example error codes:

- `UNSUPPORTED_OPERATION` — request is outside the known operation set;
- `INSPECTION_FAILED` — artifact/request cannot pass structural validation;
- `BASELINE_MISMATCH` — target precondition does not match;
- `CONFIRMATION_EXPIRED` — pending authorization is stale;
- `INVALID_OPERATION_STATE` — requested transition is not valid from current state;
- `BACKUP_FAILED` — recovery checkpoint was not created/verified;
- `APPLY_FAILED` — mutation step failed before successful verification;
- `VERIFICATION_FAILED` — post-change checks failed and recovery is required;
- `RECOVERY_FAILED` — rollback/recovery could not be verified.

## Idempotency semantics

A duplicate confirmation for the same `operation_id` behaves according to stored state:

| Current state | Duplicate confirmation result |
|---|---|
| `pending_confirmation` | may execute once |
| `completed` | return existing completed result; do not apply again |
| `rolled_back` | return existing rollback result; do not apply again |
| `expired` / `rejected` | reject without side effect |
| `applying` / `verifying` | report in-progress state; do not start a parallel duplicate |

## Why the contract is narrow

The interface models business operations and state transitions rather than arbitrary shell commands. This makes authorization, retries, observability and testing possible at the operation level.
