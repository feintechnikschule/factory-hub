# 10. Interface contract baseline

Status: Proposed; detailed bindings remain undecided.

## Context

R8 requires well-defined service interfaces. ADR 0006 establishes protocol boundaries but leaves topic names, payload schemas, the OPC UA information model, MQTT quality-of-service settings and internal service-to-service transport open. ADR 0007 further limits OPC UA and MQTT to local software interfaces; neither is assumed on physical equipment.

The hub repository contains no component implementations or verified equipment signals from which to finalize those bindings. This ADR records a candidate baseline for component-repository design and identifies what still needs agreement. The candidates below are not implementation contracts.

## Proposal

- Two contracts replace the three-level chain adapter - proxy - edge gateway - twin/MES:
  1. Adapter - proxy: local to each machine. The proxy starts a service with `start(service, parameters)`; the equipment (or the simulation) answers `OK` or `ERR` on start and again on finish, with a status container. Simulation implements the same contract, so nothing above the adapter changes.
  2. Proxy - MQTT broker - digital twin and MES: proxies publish events directly to the local MQTT broker; the twin and MES subscribe. There is no separate gateway hop. This supersedes the gateway part of the MQTT boundary in [ADR 0006](0006-communication-protocol-boundaries.md); `factory-edge-gateway` is reduced to broker operation and topic namespace, or removed.
- Job handover: the winning proxy receives from the process service the service name and order parameters (product type, unique part ID, customer-set parameters). The proxy looks up the product details for the service and passes the relevant details through the adapter. The transport of this handover is internal service-to-service communication and remains undecided.
- Every start and finish event uses one fixed envelope; only `status` varies by service and machine and is an opaque container to the transport. Candidate fields: `schemaVersion`, `eventId`, `event` (`started` or `finished`), `result` (`OK` or `ERR`), `orderId`, `partId`, `service`, `equipmentId`, `observedAt` (UTC), `feedbackSource` (`physical`, `predefinedSimulation`, or `operatorSimulation`), `status`. Preserve the feedback provenance required by [ADR 0008](0008-adapter-operational-modes.md) and the observation time required by [ADR 0003](0003-soft-real-time-timing-policy.md).
- Candidate upstream topics: `factory/v1/equipment/{equipmentId}/events` (started/finished) and `factory/v1/equipment/{equipmentId}/state` (status). `{equipmentId}` identifies one stable machine or transport proxy. These publications report feedback; they are not commands or proof that a command succeeded.
- Return path for simulation: when the digital twin runs in operator-driven simulation, button and checkbox actions are published to `factory/v1/equipment/{equipmentId}/sim/cmd`. Each proxy subscribes only to its own command topic. Candidate command fields: `commandId`, `equipmentId`, `control`, `value`, `source`, `issuedAt`.
  - The controls per machine are a fixed set defined by the machine design; the twin does not need them announced dynamically.
  - Only the simulation adapter acts on these commands. In physical mode the proxy neither subscribes nor acts, so a twin click can never operate a physical machine.
  - The twin does not update its display on click. The adapter applies the command, the proxy publishes the resulting state, and the twin renders that state.
  - Commands are not retained, so they are not replayed on reconnect; receivers ignore a repeated `commandId`.
- The MES does not send commands. It only consumes published events and state.
- Broker access rules (candidate): the twin may publish only to `*/sim/cmd`; each proxy may publish only to its own events and state topics and subscribe only to its own command topic; the MES is subscribe-only.
- Use MQTT QoS 1 as a candidate for events, state and simulation commands. Consumers must tolerate duplicate delivery, correlate by equipment and event identity, and reject feedback that exceeds the configured age in [ADR 0003](0003-soft-real-time-timing-policy.md). QoS 1 is not exactly-once and does not establish a command or completion guarantee.
- OPC UA: the gateway-owned virtual equipment model in ADR 0006/0007 is not needed for this event flow. Whether any OPC UA address space is still required (for example for local software clients) is undecided.
- Do not infer a transport for internal service-to-service interactions from the MQTT boundary. Any selected transport must support the job/round correlation, idempotent handling, participant acknowledgements and absolute deadline required by [ADRs 0003](0003-soft-real-time-timing-policy.md) and [0004](0004-fail-closed-machine-negotiation.md).

## Still undecided

The following items require agreement in the relevant component repositories before implementation relies on them:

- Whether to adopt the candidate topic pattern, exact equipment identifiers, topic versioning and broker access rules.
- The concrete payload schema for each asset type, including field names, types, units, enumerations, null/error semantics and compatibility policy.
- The MQTT QoS, retained-message behavior, session settings and delivery guarantees for each message class.
- Whether an OPC UA address space is still needed and, if so, its namespace URI, node identifiers and types.
- Per-machine status container contents and the fixed control set of each machine.
- The transport and exact interaction contracts for service-to-service negotiation, commands, acknowledgements and results.

Physical command and feedback mappings remain subject to the [equipment interface inventory](../architecture/equipment-interface-inventory.md) (ADR 0007). Nothing in this proposal changes the safety boundary or makes simulated feedback equivalent to physical feedback.

## Consequences

- Component teams have a common candidate shape to evaluate without treating unverified details as settled.
- The interface-contract open point remains open until the relevant component owners approve and specify the bindings.
- Cross-component implementation must not depend on these candidate names, fields or model nodes until that agreement is recorded.
