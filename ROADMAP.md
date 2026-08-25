# BullADM roadmap

The public case evolves by adding analysis evidence, not by publishing privileged implementation details.

## Now

### Operation audit evidence — [Issue #1](https://github.com/blsoo/bulladm-ops-automation/issues/1)

Define the minimum evidence needed for every terminal operation state.

Questions:
- which fields prove baseline, authorization, apply and verification?;
- how should rollback evidence differ from ordinary failure evidence?;
- how do retries remain idempotent without hiding repeated delivery attempts?;
- which sensitive fields must never enter audit payloads?

## Next

- map risk-register items to requirements/tests explicitly;
- add a recovery-time / verification-time requirement model;
- add a change-impact example for introducing a second operation type;
- model authorization roles without exposing production identity details;
- add conceptual observability/metrics contract for operation outcomes.

## Later

- formalize operator-facing status/error messages;
- model partial dependency outage scenarios;
- document incident escalation after failed recovery;
- extend CI checks for additional public-safe artifact types.

## Completion rule

A new operational capability is not "done" because it can execute. The analysis should answer:

1. What exact state change is allowed?
2. What baseline makes it safe?
3. Who/what authorizes it?
4. What recovery point exists?
5. How is success verified?
6. What happens on ambiguous outcome?
7. What evidence remains afterward?
