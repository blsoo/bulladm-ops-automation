# BullADM diagrams

Public-safe diagrams for the operational control plane. They deliberately omit real credentials, hostnames, addresses, privileged paths and private endpoints.

## 1. Control-plane context

```mermaid
flowchart LR
    OP[Operator] --> BOT[Telegram Admin Interface]
    BOT --> GW[Authenticated Control Gateway]
    GW --> OPS[Operational Service]
    OPS --> INSPECT[Inspect / Preflight]
    OPS --> BACKUP[Backup Service]
    OPS --> APPLY[Apply Change]
    OPS --> VERIFY[Verification]
    OPS --> AUDIT[Audit / Status]
    APPLY --> APP[Application State]
    BACKUP --> REC[(Recovery Checkpoint)]
    VERIFY --> APP
```

Telegram is an operator interface, not an unrestricted shell.

## 2. Deployment workflow

```mermaid
flowchart TD
    A[Artifact received] --> B[Inspect]
    B --> C{Inspection passed?}
    C -- No --> X[Reject]
    C -- Yes --> D[Preflight checks]
    D --> E{Baseline valid?}
    E -- No --> Y[Stop: ambiguous state]
    E -- Yes --> F[Create pending operation]
    F --> G[Show operator preview]
    G --> H{Explicit confirmation?}
    H -- No --> Z[Cancel / expire]
    H -- Yes --> I[Create backup]
    I --> J{Backup succeeded?}
    J -- No --> Y
    J -- Yes --> K[Apply minimal change]
    K --> L[Verify]
    L --> M{Verification passed?}
    M -- Yes --> N[Success]
    M -- No --> R[Rollback]
    R --> RV[Verify recovery]
```

## 3. Operation state machine

```mermaid
stateDiagram-v2
    [*] --> received
    received --> inspected
    received --> rejected: malformed
    inspected --> preflight_passed
    inspected --> rejected: inspection failed
    preflight_passed --> pending_confirmation
    pending_confirmation --> expired
    pending_confirmation --> backup_creating: confirmed
    backup_creating --> failed: backup failed
    backup_creating --> applying: backup ready
    applying --> verifying
    applying --> failed: apply failed
    verifying --> completed: checks pass
    verifying --> rollback: checks fail
    rollback --> rolled_back: recovery verified
    rollback --> failed: recovery failed
    completed --> [*]
    rejected --> [*]
    expired --> [*]
    rolled_back --> [*]
```

## 4. Confirmation sequence

```mermaid
sequenceDiagram
    actor Operator
    participant UI as Admin UI
    participant Gateway as Auth Gateway
    participant Ops as Operation Service
    participant Backup
    participant App as Application State

    Operator->>UI: submit artifact/action
    UI->>Gateway: authenticated request
    Gateway->>Ops: inspect + preflight
    Ops-->>Operator: preview pending operation
    Operator->>UI: explicit confirm
    UI->>Gateway: confirm operation id
    Gateway->>Ops: execute confirmed operation
    Ops->>Backup: create checkpoint
    Backup-->>Ops: ready
    Ops->>App: apply change
    Ops->>App: verify
    alt verification passes
        Ops-->>Operator: success
    else verification fails
        Ops->>Backup: restore checkpoint
        Ops->>App: verify recovery
        Ops-->>Operator: rolled back / failure status
    end
```

## 5. Trust boundaries

```mermaid
flowchart LR
    subgraph OperatorSide[Operator side]
        O[Operator]
        T[Telegram Client]
    end

    subgraph ControlPlane[Control plane]
        B[Admin Bot]
        G[Authenticated Gateway]
        S[Operational Service]
    end

    subgraph Application[Application boundary]
        A[Application]
        R[(Recoverable State)]
    end

    O --> T --> B
    B --> G --> S
    S --> A
    S --> R
```

The security idea is narrow authority: the chat interface can request known operations, but it does not expose generic privileged command execution.

## 6. Failure decision tree

```mermaid
flowchart TD
    A[Operation requested] --> B{Known operation?}
    B -- No --> STOP[Stop]
    B -- Yes --> C{Expected baseline?}
    C -- No --> STOP
    C -- Yes --> D{Fresh confirmation?}
    D -- No --> STOP
    D -- Yes --> E{Backup available?}
    E -- No --> STOP
    E -- Yes --> F[Apply]
    F --> G{Verification pass?}
    G -- Yes --> OK[Complete]
    G -- No --> H[Rollback]
    H --> I{Recovery verified?}
    I -- Yes --> RB[Report rolled back]
    I -- No --> INC[Escalate incident]
```

## 7. Idempotent retry flow

```mermaid
flowchart TD
    A[Confirm operation] --> B{Current operation state}
    B -- pending --> C[Execute once]
    C --> D[Store terminal result]
    D --> E[Return result]
    B -- completed --> F[Return stored success]
    B -- rolled_back --> G[Return stored rollback result]
    B -- expired/rejected --> H[Reject without side effect]
```

## Interview talking points

- Why an admin bot should not become a remote shell.
- Why confirmation and execution are separate operations.
- Why baseline validation prevents applying a valid artifact to the wrong state.
- Why backup creation is a hard precondition rather than a best-effort step.
- How state machines make operational behaviour auditable.
- How fail-closed design changes error handling for privileged workflows.
