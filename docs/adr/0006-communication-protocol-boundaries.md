# 6. Communication protocol boundaries

Status: Accepted; equipment-facing OPC UA boundary superseded by [ADR 0007](0007-local-machine-proxies-and-legacy-adapters.md).

## Context

This ADR originally recorded OPC UA at the equipment/gateway boundary and MQTT between the gateway, `factory-mes-service` and `factory-digital-twin`. Requirement R8 calls for well-defined service interfaces, but the repository does not specify their detailed contracts. The logical component diagram also did not assign transports to communication between services hosted on the central computer.

## Decision

The original protocol direction was:

- OPC UA is the equipment-facing protocol between machines and `factory-edge-gateway` (superseded by ADR 0007).
- MQTT carries gateway data from `factory-edge-gateway` to `factory-mes-service` and `factory-digital-twin`.

The MQTT data-distribution boundary remains. ADR 0007 defines the current local-software use of OPC UA and MQTT.

The protocol boundaries are not a complete interface specification. This ADR does not prescribe internal service-to-service transports, topic names, payload schemas, OPC UA information models, MQTT quality-of-service settings, or delivery guarantees. Those details remain open and must be specified by the component repositories before implementation relies on them.

The logical interactions in the architecture overview describe responsibilities and information flow; they do not imply that every interaction uses OPC UA or MQTT.

## Consequences

- The original equipment-facing OPC UA boundary is historical and superseded; equipment-specific integration is defined in ADR 0007.
- The MES and twin consume gateway-published data rather than depending directly on equipment protocols.
- The protocol names alone do not satisfy R8. Concrete interface contracts and their operational guarantees still need specification in the relevant component repositories.
- Internal communication between services on the central computer remains undecided.
