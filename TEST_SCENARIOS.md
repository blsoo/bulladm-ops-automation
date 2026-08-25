# BullADM — system test scenarios

These black-box scenarios verify the public workflow behaviour without depending on real infrastructure.

| ID | Scenario | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TS-01 | Inspect valid operation | Supported synthetic artifact/request | Submit for inspection | Inspection passes; no target mutation occurs |
| TS-02 | Reject malformed input | Invalid structure | Submit for inspection | Rejected before preflight/apply |
| TS-03 | Baseline match | Target baseline matches expected value | Run preflight | Pending operation may be created |
| TS-04 | Baseline mismatch | Target baseline differs | Run preflight | Operation stops; no confirmation/apply path |
| TS-05 | Missing confirmation | Valid pending operation | Do not confirm | No mutation; operation remains pending until expiry/cancel |
| TS-06 | Expired confirmation | Pending operation is expired | Confirm | Rejected without mutation |
| TS-07 | Backup failure | Valid confirmed operation; checkpoint cannot be created | Execute | Apply does not start; failure stage = backup |
| TS-08 | Successful apply | Valid confirmed operation; backup succeeds | Execute + verify | State becomes completed only after checks pass |
| TS-09 | Verification failure | Apply completes but checks fail | Continue workflow | Rollback branch starts |
| TS-10 | Successful rollback | Verification failed; checkpoint valid | Restore + verify recovery | State becomes rolled_back |
| TS-11 | Recovery failure | Verification failed; recovery cannot be verified | Restore + verify | State becomes failed/escalated, not rolled_back |
| TS-12 | Duplicate confirmation after success | Operation already completed | Confirm same id again | Existing result returned; no second apply |
| TS-13 | Duplicate while applying | Operation currently applying/verifying | Confirm same id | In-progress status returned; no parallel duplicate |
| TS-14 | Changed scope after preview | Pending operation exists | Attempt confirmation with different target parameters | Stored operation remains authoritative; replacement scope is rejected/ignored |
| TS-15 | Unsupported generic command | Request is not a known bounded operation | Submit | Rejected; interface does not turn into remote shell |

## Detailed scenario — TS-04 baseline mismatch

**Goal:** prove that a syntactically valid operation is not enough to authorize mutation.

**Preconditions**

- inspection passes;
- operation expects baseline `A`;
- target currently represents baseline `B`.

**Steps**

1. Run preflight.
2. Read operation state.
3. Attempt to continue to confirmation/apply.

**Expected**

- preflight reports baseline mismatch;
- no executable pending operation is produced;
- no recovery checkpoint is created unnecessarily;
- no target mutation occurs.

## Detailed scenario — TS-12 duplicate confirmation

**Goal:** verify one operation produces one business effect even when the client retries after a timeout.

**Preconditions**

- operation was confirmed and completed successfully;
- client did not receive/retain the first response.

**Steps**

1. Submit the same confirmation again.
2. Query current operation state.
3. Compare target mutation history/effect count.

**Expected**

- target is not mutated again;
- state remains `completed`;
- stored successful result can be returned;
- no second recovery checkpoint/apply sequence starts.

## Detailed scenario — TS-09/TS-10 recovery path

**Goal:** verify that rollback is part of normal workflow design.

**Preconditions**

- valid checkpoint exists;
- apply step finished;
- post-apply verification fails.

**Steps**

1. Observe verification failure.
2. Enter rollback state.
3. Restore checkpoint.
4. Run recovery verification.

**Expected**

- operation never reports `completed`;
- rollback uses the operation's known checkpoint;
- only verified recovery results in `rolled_back`;
- failed recovery produces a distinct failure/escalation outcome.
