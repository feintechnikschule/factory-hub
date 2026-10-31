# 9. Reset to default state on restart

Status: Accepted

## Context

All services run on one central computer, which is a physical single point of failure. A crash can interrupt an order while a physical machine is running. The physical machines are legacy equipment: after startup their only confirmed state is the default state. The machine proxy (ADR 0007) is the only place where a confirmed state beyond the default exists. If a machine keeps running during a crash, its real state can evolve and the proxy's last confirmed state becomes invalid. ADR 0003 requires that recovery uses confirmed state, never assumptions.

## Decision

**Recovery rule for a machine proxy after a crash or restart**

- The proxy recovers its last confirmed state and continues only if the physical machine is verifiably in its default state and not running.
- In every other case the physical machine is restarted to its default state and the proxy is reset to its default state. This includes the case where default state or idle cannot be verified: unknown means reset.

**Transport systems**

- Restarting a transport system to its default state sends all its carriers to their home positions.
- Every other physical machine with a carrier at its station on that transport system loses its workpiece position and is restarted too, and its proxy is reset.
- The assignment of machines to transport systems is configured, so the cascade is deterministic.

**Orders**

- When a machine proxy is reset, the partly manufactured product of its current job is lost.
- The process service (one instance per order) containing that machine's job is reset, and the order starts over from the beginning.
- Only orders whose product was on a reset machine or carrier are affected. Other orders continue.
- The lost workpiece still exists physically. The operator clears it and confirms before the order is started over, because a new workpiece is needed.

**Negotiation**

- Negotiation state is in memory only. After a restart, uncommitted rounds and reservations (ADR 0004) are void.

**Simulation**

- In simulation mode (ADR 0008) the same rules apply to the simulated machines. The adapter starts from the configured scenario or the digital-twin controls, and that state is never presented as physical confirmation.

Safety interlocks and emergency stops remain in the existing control equipment and are never bypassed by recovery.

## Open points

- **Persistence:** recovering a proxy's confirmed state requires that this state survives the crash. Whether and where it is persisted is not decided. Without persistence every restart resets all proxies and orders.
- **Default and idle detection:** the adapter must be able to read "default state and not running" from each physical machine. Whether this is possible per machine has to be established in the equipment interface inventory (ADR 0007). Machines that cannot report it are always reset.

## Consequences

- No double treatment or double movement results from replaying commands, and no stale state is trusted.
- A crash during production usually costs the affected orders and requires manual clearing of workpieces.
- Availability is limited by the single computer. This ADR defines safe recovery, not redundancy or failover.
- Transport-system resets can cascade to every machine on that system.
