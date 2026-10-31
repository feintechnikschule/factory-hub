# 4. Fail-closed machine negotiation

Status: Accepted

## Context

ADR 0002 selects a machine by having machine services exchange status, cost and speed, then independently apply a deterministic scoring rule. That rule selects one winner only if every service uses the same complete set of offers. Delayed, missing, duplicated or inconsistent messages can otherwise leave services with different views, allowing stale offers or more than one apparent winner.

The project requires decentralized machine selection and has no separate coordinator for that decision. The production-process service announces the job and enforces its deadline, but must not choose a machine.

## Decision

Machine negotiation fails closed: a machine may execute a treatment job only after every machine service in that job's announced participant set has confirmed the same complete offer set and the same winner.

For each job, negotiation proceeds in rounds identified by the job ID and round number:

1. The production-process service announces the job, participant set and absolute deadline. The participant set remains fixed for that job.
2. Each participant publishes one offer for the round, including its current eligibility, selection inputs and feedback timestamp. Busy or unable machines publish an ineligible offer rather than silently leaving the participant set. Offers are bound to the job and round.
3. A participant evaluates a round only after receiving one valid offer from every participant. Offers whose feedback exceeds the configured maximum age in ADR 0003 are invalid. Each participant independently applies ADR 0002's deterministic scoring and tie-break rule to that identical set.
4. A participant acknowledges readiness only if it has the complete valid offer set and computes the same winner. The selected machine must also atomically reserve its capacity for the job before acknowledging; it must not reserve capacity already committed to overlapping work.
5. The selected machine may commit and command its equipment only after it has readiness acknowledgements from every participant. It then reports the committed assignment to the production-process service.

Missing, duplicate-conflicting, expired or inconsistent round data prevents readiness; it never creates an implicit winner. An uncommitted reservation is released when the round aborts or the job deadline expires. If the selected machine cannot reserve its capacity, the round aborts and a new round may be tried with updated offers, only within the original job deadline. A participant that cannot take part prevents a commit; there is no quorum rule or centralized fallback.

Messages are scoped to job and round so delayed or duplicate messages cannot be applied to a later round. Services reject a commit after the job deadline. Once a job is committed, later rounds for that job are invalid.

## Consequences

- At most one service can commit a job when participants follow the protocol, even if messages are delayed or views differ; disagreement prevents progress rather than permitting competing assignments.
- Negotiation requires participation from every service in the announced participant set. A failed or unreachable service can therefore prevent the job from being assigned before its deadline.
- The protocol does not guarantee progress under communication failures. Timed-out work is handled under ADR 0003, and recovery must be based on confirmed equipment state.
- Machine services need job/round correlation, reservation handling, and idempotent message processing.
