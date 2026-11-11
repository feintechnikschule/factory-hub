# 11. Persist confirmed proxy state

Status: Accepted

## Context

ADR 0009 permits a machine proxy to recover its last confirmed state after a restart only when the physical machine is verifiably in its default state and not running. Otherwise, or when that condition cannot be verified, the machine and proxy are reset. The proxy therefore needs durable state across a process or central-computer restart. MQTT delivery and in-memory state are not durable records of the proxy's confirmed state.

## Decision

- Each machine proxy owns a durable local store on the central computer for its latest confirmed state. The broker and remote services are not the source of truth for recovery.
- Persist a complete snapshot atomically after each confirmed state transition and before reporting that transition as successful to other services. The snapshot includes the stable equipment identity, state, associated order/part/service when present, feedback source, observation time, and a schema version. It does not include pending commands, unconfirmed transitions, or negotiation rounds.
- If the snapshot cannot be committed, the proxy must not report the transition as successful or accept further work for that machine. It treats the machine state as uncertain and requires reconciliation or the reset procedure in ADR 0009.
- On restart, a missing, invalid, incompatible, or unreadable snapshot is treated as unavailable state. The proxy applies ADR 0009's fail-closed reset rule; it never reconstructs state from broker messages or replays commands.
- A valid snapshot is eligible for recovery only after the adapter verifies that the physical machine is in its default state and not running, as required by ADR 0009. Unknown verification results require reset.

The storage technology is an implementation choice, but it must provide durable writes and atomic snapshot replacement. This decision covers recovery from process or computer restarts, not central-computer failure or redundancy.

## Consequences

- A completed transition is recoverable only after its confirmed snapshot is durable.
- A crash between an equipment action and durable persistence can require resetting the machine and restarting the affected order; unconfirmed progress is never assumed.
- Proxies remain unavailable for new work when durable persistence fails, rather than continuing with state that cannot be recovered safely.
- The central computer remains a single point of failure; this ADR does not provide replication or failover.
