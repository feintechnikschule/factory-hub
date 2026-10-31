# 2. Decentralized machine selection by negotiation

Status: Proposed

## Context

The factory has two milling machines (each with its own transport system). Workpieces and carriers are passive. The machines are not Industrie 4.0 ready, so all logic runs as logically independent services on one central computer, connected to the existing controls through equipment adapters (see `docs/concepts/industrie-4-0-principles.md`, requirements R4, R5, R7).

For each treatment step, one machine must be selected. Two options were considered:

1. **Centralized selection (offer-and-select):** machine services publish status, cost and speed; the production-process service selects. This is automated but centralized.
2. **Decentralized selection (negotiation):** the machine services negotiate among themselves who takes the job.

The purpose of this project is to demonstrate decentralized decisions, so option 2 is chosen despite its extra complexity.

## Decision

Machine selection is **decentralized**. Decision authority is distributed as follows:

| Decision | Owner |
|---|---|
| Which machine takes a treatment job | Machine services, by negotiation |
| Whether the machine starts and performs the treatment | The machine service that took the job |
| Whether a product cannot be manufactured | Production-process service |

### Protocol

1. The production-process service (one instance per order) announces the treatment job to all machine services. The announcement carries the job parameters, including the customer preference (speed over cost, or cost over speed) and a deadline.
2. Each machine service computes a score from its own status, cost and speed, weighted by the customer preference, and exchanges it with the other machine services.
3. All machine services apply the same deterministic rule: the best eligible score wins; ties are broken by machine ID. Busy or unable machines report themselves as ineligible. This rule selects a winner only when services use the same complete offer set; the fail-closed agreement protocol in [ADR 0004](0004-fail-closed-machine-negotiation.md) governs when that winner may act.
4. The winning machine service commits only after every participant confirms the same winner, then reports "job taken" to the production-process service, treats the workpiece through its adapter, and reports the confirmed result (see [ADR 0004](0004-fail-closed-machine-negotiation.md)).
5. If no machine takes the job before the deadline, the production-process service withdraws the announcement and flags the product as not manufacturable. The deadline is authoritative; retries do not extend it and there is no separate retry limit (see [ADR 0003](0003-soft-real-time-timing-policy.md)).

Several machines wanting the job is normal contention and is resolved by the rule in step 3. It is not a failure. There is no fallback to a central decision by the production-process service.

## Consequences

- Machine services can be added or removed without changing the production-process service or any selection logic (R7), as long as they follow the protocol.
- The protocol adds failure modes that must be handled: livelock, stale scores, double assignment, and timeouts. [ADR 0004](0004-fail-closed-machine-negotiation.md) defines fail-closed agreement for stale or inconsistent views and double assignment.
- The per-process negotiation deadline and other timing budgets must be configured and validated against the target installation as described in [ADR 0003](0003-soft-real-time-timing-policy.md).
- The decentralization is logical only. All services run on one computer, which remains a single point of failure.
- Safety interlocks and emergency stops stay in the existing control equipment.
- If the negotiation proves too complex or unreliable, the centralized offer-and-select alternative remains available. Superseding this ADR would be required.
