# Project status and hand-off

Start here when resuming work. Update this file when decisions or open points change.

## Reading order

1. `docs/concepts/industrie-4-0-principles.md` – principles, requirements R1–R8, constraints
2. `docs/architecture/overview.md` – diagrams (protocols, adapter modes, components, sequence)
3. `docs/architecture/implementation-proposal.md` – agreed simulated vertical-slice sequence
4. `docs/adr/` – decisions below

## Decisions (ADRs)

| ADR | Decision |
|---|---|
| 0001 | Record architecture decisions as ADRs |
| 0002 | Decentralized machine selection by negotiation between machine services |
| 0003 | Soft-real-time timing policy: configured budgets, one authoritative deadline, no retry limit, no numeric defaults |
| 0004 | Fail-closed negotiation: a machine commits only after all participants confirm the same complete offer set and winner |
| 0005 | Failure diagnostics deferred: "not manufacturable" stays generic until simulation and real-world tests show what is needed |
| 0006 | Protocol boundaries (equipment-facing OPC UA part superseded by 0007; OPC UA and gateway hop dropped in 0010) |
| 0007 | Machine services are software proxies on the central computer; adapters connect them to legacy machines; MQTT is a local software interface only (OPC UA dropped in 0010) |
| 0008 | Adapter modes: physical machine connected, or simulation (predefined scenario or operator-driven digital twin); simulated feedback is never physical confirmation |
| 0009 | Reset to default state on restart; recover confirmed state only if the machine is in default state and not running; transport reset cascades to machines with a carrier at their station; reset proxy means the order starts over |
| 0010 | Proposed interface contract baseline: MQTT only, OPC UA dropped; two contracts (adapter - proxy; proxy - MQTT broker - twin/MES), no gateway hop, fixed event envelope with variable status container, simulation commands from twin to proxy via per-machine MQTT topic, MES sends no commands; detailed bindings remain undecided |
| 0011 | Atomic durable snapshots before reporting confirmed transitions; persistence failures stop further work; storage and coordinated order recovery remain undecided |

## Open points

- **Persistence implementation** (ADR 0011): durable confirmed-state snapshots and fail-closed persistence are required, including in the simulated slice. Storage technology, retention and migration, and coordination with process/order recovery remain undecided.
- **Default and idle detection** (ADR 0009): whether each real machine can report "default state and not running". Needs field verification; see the equipment interface inventory.
- **Equipment field verification** (ADR 0007): a preliminary inventory of documented equipment and field-verification questions is in [the inventory](architecture/equipment-interface-inventory.md). Actual controls, sensors and controller interfaces remain unverified; choose adapter technology only after inspection.
- **Timing values** (ADR 0003): budgets are measured and configured per installation.
- **Interface bindings** (R8, ADRs 0006 and 0010): the candidate baseline (direct proxy-to-broker MQTT, no gateway hop; simulation command topic per machine; MES subscribe-only) is documented. Topic names, payload schemas, QoS/delivery settings, broker access rules, and the internal service-to-service transport (including job handover from the process service) still require agreement. Also decide whether `factory-edge-gateway` is removed or reduced to broker operation.
- **Failure diagnostics** (ADR 0005): revisit after simulation and real-world tests.
- **Requirements** (`docs/requirements/`): still empty. R1–R8 live only in the principles doc.
- **Concepts** (`docs/concepts/`): README is a stub (MQTT, digital twin).
- **Implementation direction:** the simulated single-order vertical slice and follow-on sequence are agreed in [the implementation proposal](architecture/implementation-proposal.md). Code will live in `feintechnikschule/factory-implementation`.

## Working conventions

- Decisions go in `docs/adr/` as the next numbered ADR; use Mermaid for diagrams.
- New or edited Mermaid flowcharts use top-down `flowchart TD`, not left-right `flowchart LR`.
- Changes go through pull requests on a branch.
