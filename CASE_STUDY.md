# BullADM — operational automation case study

BullADM is an administrative control-plane concept built to reduce the risk and friction of operating a live service from a phone.

The interesting part is not the bot UI itself. The project is about turning dangerous operational actions into a controlled workflow with explicit verification, confirmation and rollback.

## Problem

Directly editing production files or running long server commands from a phone is error-prone. A safer workflow should make the intended change reviewable before it reaches production and preserve a recovery path if validation fails.

BullADM is designed around this sequence:

```text
artifact received
      -> inspect
      -> preflight checks
      -> explicit operator confirmation
      -> backup
      -> apply minimal change
      -> verification
      -> success / rollback path
```

## Confirmation-gated operations

High-impact actions are intentionally not one-click mutations. The system distinguishes between preparing an operation and authorizing it.

This reduces the chance that an accidental button press or malformed request directly changes live state.

## Backup before mutation

Rollback is part of deployment design rather than an emergency feature added later.

A change is not considered safe merely because it can be applied. The operator should also know:

- what baseline is expected;
- what scope will change;
- whether a recovery checkpoint exists;
- how verification will be performed;
- what the rollback path is.

## Authenticated control gateway

Administrative requests pass through an authenticated gateway rather than exposing unrestricted shell access through Telegram.

The public case deliberately omits concrete secrets and infrastructure details.

## Mobile-first operations

The workflow is designed for cases where the operator is away from a computer. That influences the UX:

- short inspectable operations;
- clear pending/confirmed states;
- explicit confirmation for destructive steps;
- status messages understandable from a phone;
- rollback that does not require reconstructing the previous state manually.

## Separation of control plane and application

The administrative workflow is treated as a separate control plane rather than mixing deployment logic into the public application.

That creates a cleaner boundary between:

- application state;
- operational commands;
- authorization;
- recovery checkpoints;
- audit and verification.

## Failure modes considered

The workflow stops rather than guesses when:

- the expected baseline does not match;
- an artifact fails inspection;
- confirmation is missing or stale;
- backup creation fails;
- post-apply verification fails;
- required state is ambiguous.

The general rule is **fail closed for operational uncertainty**.

## Example interview discussion

**Why not just SSH into the server and edit the file?**

Direct access may be faster for one isolated change, but it has poor repeatability and weak safeguards. A controlled workflow gives the change a known baseline, inspection step, explicit authorization, recovery checkpoint, verification and rollback path.

**Why use Telegram at all?**

Telegram is only the operator interface. The important engineering decision is that the chat client does not become an unrestricted shell; it invokes a narrow set of authenticated operational actions with safety gates.

## What this project demonstrates

- operational automation;
- deployment workflow design;
- confirmation gates;
- authenticated service-to-service control;
- backup/rollback thinking;
- mobile-first admin UX;
- fail-closed behaviour;
- separation of application and control plane;
- production-safety mindset.
