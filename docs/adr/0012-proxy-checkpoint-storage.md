# 12. Proxy checkpoint storage

Status: Accepted

## Context

[ADR 0011](0011-persistent-proxy-state.md) requires each equipment proxy to keep a durable, atomic checkpoint of its latest confirmed state. Checkpoints must survive proxy and host restarts, but the hub has no component implementation, storage constraints or disk-failure protection requirement. A broker message or in-memory record is not a durable checkpoint.

## Decision

- Store each equipment proxy's checkpoint in a file-backed SQLite database on the central computer's local persistent filesystem. The database is local to the proxy and is not shared with the broker or other services.
- Use a transaction to replace the complete checkpoint for an equipment identity. Configure SQLite with rollback-journal mode and `synchronous=FULL`; do not acknowledge a state transition until the transaction commits successfully.
- Keep only the latest checkpoint per equipment identity. Historical state and backup/replication are outside this decision; the storage does not protect against disk loss or host failure.
- The database and its journal must reside together on a local filesystem that supports SQLite locking and synchronization. Do not place checkpoint databases on temporary or network filesystems.
- Version the database schema and checkpoint record. Apply schema migrations transactionally and monotonically. If the database is corrupt, a migration fails, or the schema or record version is unsupported, treat the checkpoint as unavailable and follow ADR 0009's fail-closed reset procedure. Do not silently downgrade or reconstruct state from broker messages.
- A failed database write makes the checkpoint unavailable for recovery. The proxy must not report the transition as committed or accept further work on the basis of an unpersisted state.

This selects the persistence mechanism for proxy checkpoints only. It does not define persistence or atomic coordination of process/order state; those require a separate decision and must not be inferred from this database.

## Consequences

- SQLite provides transactional checkpoint replacement without introducing a network database service.
- Deployments must provision persistent local storage and preserve the database and journal across host restarts.
- Unsupported migrations or storage failures can require equipment reset and order restart under ADR 0009.
- Process/order recovery remains unresolved; a proxy checkpoint alone does not authorize an order to resume.
