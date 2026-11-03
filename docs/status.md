# Project status and hand-off

Start here when resuming work. Update this file when decisions or open points change.

## Reading order

1. `docs/concepts/industrie-4-0-principles.md` – principles, requirements R1–R8, constraints
2. `docs/architecture/overview.md` – diagrams (protocols, adapter modes, components, sequence)
3. `docs/adr/` – decisions below

## Decisions (ADRs)

| ADR | Decision |
|---|---|
| 0001 | Record architecture decisions as ADRs |
| 0002 | Decentralized machine selection by negotiation between machine services |
| 0003 | Soft-real-time timing policy: configured budgets, one authoritative deadline, no retry limit, no numeric defaults |
| 0004 | Fail-closed negotiation: a machine commits only after all participants confirm the same complete offer set and winner |
| 0005 | Failure diagnostics deferred: "not manufacturable" stays generic until simulation and real-world tests show what is needed |
| 0006 | Protocol boundaries (equipment-facing OPC UA part superseded by 0007) |
| 0007 | Machine services are software proxies on the central computer; adapters connect them to legacy machines; OPC UA and MQTT are local software interfaces only |
| 0008 | Adapter modes: physical machine connected, or simulation (predefined scenario or operator-driven digital twin); simulated feedback is never physical confirmation |
| 0009 | Reset to default state on restart; recover confirmed state only if the machine is in default state and not running; transport reset cascades to machines with a carrier at their station; reset proxy means the order starts over |

## Open points

- **Persistence of proxy state** (ADR 0009): where the confirmed state is stored so it survives a crash. Undecided.
- **Default and idle detection** (ADR 0009): whether each real machine can report "default state and not running". Needs field verification; see the equipment interface inventory.
- **Equipment interface inventory** (ADR 0007): a preliminary inventory of documented equipment and field-verification questions is in [the inventory](architecture/equipment-interface-inventory.md). Actual controls, sensors and controller interfaces remain unverified; choose adapter technology only after inspection.
- **Timing values** (ADR 0003): budgets are measured and configured per installation.
- **Interface contracts** (R8, ADR 0006): topic names, payload schemas, OPC UA information model, MQTT QoS, internal service-to-service transport. Undecided.
- **Failure diagnostics** (ADR 0005): revisit after simulation and real-world tests.
- **Requirements** (`docs/requirements/`): still empty. R1–R8 live only in the principles doc.
- **Concepts** (`docs/concepts/`): README is a stub (OPC UA, MQTT, digital twin).

## Working conventions

- Decisions go in `docs/adr/` as the next numbered ADR; use Mermaid for diagrams.
- Changes go through pull requests on a branch.
