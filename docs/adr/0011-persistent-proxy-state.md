# 11. Persistent confirmed proxy state

Status: Accepted; coordinated order recovery remains undecided.

## Context

[ADR 0009](0009-reset-to-default-state-on-restart.md) allows a proxy to recover its last confirmed state after a restart, subject to independently verifying that the physical machine is in its default state and not running. Without persistence, that state is lost on every restart. The hub has no component implementation or storage constraints from which to select a persistence technology.

## Decision

- Each equipment proxy (machine or transport system) maintains a durable checkpoint of its most recently confirmed state, keyed by stable equipment identity. A checkpoint records the state, observation time, feedback source and record version.
- Only confirmed equipment or simulation feedback may update the checkpoint. A command or requested state is never persisted as confirmed state.
- Checkpoint updates are atomic. A checkpoint is considered recoverable only after its complete update is durably committed. If an update cannot be committed, recovery must not assume it was saved.
- On startup, the proxy validates the checkpoint and treats a missing, invalid or unsupported-version record as unavailable. The checkpoint is historical evidence of the last confirmed state, not evidence of the equipment's current state.
- Recovery follows ADR 0009: the proxy may recover and continue only after independently verifying the applicable default and not-running state for its equipment. Otherwise, reset the equipment, proxy and affected order as specified there.
- This decision requires durability across a proxy process crash and host restart while the underlying storage remains intact. It does not provide disk-failure protection, redundancy or failover.
- A proxy checkpoint alone does not restore an order, negotiation, pending command or coordinated cross-service state. It does not authorize automatic order continuation.

## Still undecided

- How process/order state is persisted and coordinated with proxy recovery. Proxy storage, checkpoint retention and schema migration are specified in [ADR 0012](0012-proxy-checkpoint-storage.md).

## Consequences

- Proxy recovery has a technology-independent durability requirement without assuming that persisted state is current equipment feedback.
- Implementations must fail closed when a usable checkpoint or the independent equipment verification required by ADR 0009 is unavailable.
- Component repositories must implement the storage requirements in ADR 0012 and agree on coordinated order-recovery details before relying on automatic recovery.
