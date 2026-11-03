# 10. Interface contract baseline

Status: Proposed; detailed bindings remain undecided.

## Context

R8 requires well-defined service interfaces. ADR 0006 establishes protocol boundaries but leaves topic names, payload schemas, the OPC UA information model, MQTT quality-of-service settings and internal service-to-service transport open. ADR 0007 further limits OPC UA and MQTT to local software interfaces; neither is assumed on physical equipment.

The hub repository contains no component implementations or verified equipment signals from which to finalize those bindings. This ADR records a candidate baseline for component-repository design and identifies what still needs agreement. The candidates below are not implementation contracts.

## Proposal

- Keep the existing boundaries: the gateway publishes equipment-state updates for MES and digital-twin consumers over local MQTT; local software clients read the gateway's virtual equipment model over OPC UA.
- Treat MQTT state publications as reports of feedback, not commands or proof that a command succeeded. Preserve the feedback provenance required by [ADR 0008](0008-adapter-operational-modes.md) and the observation time required by [ADR 0003](0003-soft-real-time-timing-policy.md).
- Use `factory/v1/equipment/{equipmentId}/state` as a candidate MQTT topic pattern. `{equipmentId}` identifies one stable machine or transport proxy. A candidate JSON event envelope contains `schemaVersion`, `eventId`, `equipmentId`, `observedAt` (UTC timestamp), `feedbackSource` (`physical`, `predefinedSimulation`, or `operatorSimulation`), and `state` (equipment-specific reported values).
- Use MQTT QoS 1 as a candidate for these state publications. Consumers must tolerate duplicate delivery, correlate updates by equipment and event identity, and reject feedback that exceeds the configured age in [ADR 0003](0003-soft-real-time-timing-policy.md). QoS 1 is not an exactly-once guarantee and does not establish a command or completion guarantee.
- Model the OPC UA address space as a gateway-owned `Equipment` folder containing one object per proxy, with an object type distinguishing machines from transport systems. Expose confirmed state as variables and its observation time and feedback source as associated metadata. Do not expose simulated feedback as physical confirmation.
- Do not infer a transport for internal service-to-service interactions from the MQTT and OPC UA boundaries above. Any selected transport must support the job/round correlation, idempotent handling, participant acknowledgements and absolute deadline required by [ADRs 0003](0003-soft-real-time-timing-policy.md) and [0004](0004-fail-closed-machine-negotiation.md).

## Still undecided

The following items require agreement in the relevant component repositories before implementation relies on them:

- Whether to adopt the candidate topic pattern, exact equipment identifiers, topic versioning and broker access rules.
- The concrete payload schema for each asset type, including field names, types, units, enumerations, null/error semantics and compatibility policy.
- The MQTT QoS, retained-message behavior, session settings and delivery guarantees for each message class.
- The OPC UA namespace URI, node identifiers, object and variable types, methods/events, client assignments and compatibility policy.
- The transport and exact interaction contracts for service-to-service negotiation, commands, acknowledgements and results.

Physical command and feedback mappings remain subject to the [equipment interface inventory](../architecture/equipment-interface-inventory.md) (ADR 0007). Nothing in this proposal changes the safety boundary or makes simulated feedback equivalent to physical feedback.

## Consequences

- Component teams have a common candidate shape to evaluate without treating unverified details as settled.
- The interface-contract open point remains open until the relevant component owners approve and specify the bindings.
- Cross-component implementation must not depend on these candidate names, fields or model nodes until that agreement is recorded.
