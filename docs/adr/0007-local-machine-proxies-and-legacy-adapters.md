# 7. Local machine proxies and legacy equipment adapters

Status: Accepted; adapter operation modes are specified in [ADR 0008](0008-adapter-operational-modes.md).

## Context

The real machines are not Industrie 4.0 ready. They cannot be assumed to provide OPC UA or MQTT endpoints. The machine services in the logical architecture are software proxies running on the central computer; they represent the real machines and participate in machine selection. Equipment adapters translate between those proxies and the existing machine controls and sensors.

The architecture overview currently labels the physical machine-to-gateway boundary as OPC UA. That conflicts with the known legacy-equipment constraint. All project components also run on one computer, so OPC UA and MQTT are local software communication boundaries, not a requirement for network-capable physical machines.

## Decision

- Each machine service is a software proxy for one physical machine and runs on the central computer.
- A machine-specific equipment adapter connects each proxy to its real machine through the controls and feedback interfaces that are actually available. The adapter is responsible for translating commands and feedback; the proxy must use confirmed feedback as equipment state.
- Adapters also support disconnected simulation operation: a predefined simulated situation or operator-generated feedback through the digital twin. See [ADR 0008](0008-adapter-operational-modes.md).
- Do not assume OPC UA or MQTT exists on the physical machine. Select the adapter's concrete electrical, controller, or vendor interface only after the actual equipment interfaces have been inventoried. Safety interlocks and emergency stops remain in the existing control equipment.
- Use OPC UA as a local software interface to expose the gateway's virtual equipment model to local clients. It does not imply that the physical machines themselves speak OPC UA. The exact OPC UA information model and client assignments remain open.
- Use MQTT through a broker on the central computer for gateway-published equipment-state updates consumed by the MES and digital twin. Local clients connect to the broker; MQTT is not a direct physical-equipment connection. Topic names, payload schemas, quality-of-service settings and delivery guarantees remain open as recorded in ADR 0006.
- Local protocol endpoints may use loopback or the central computer's private container/service network, according to deployment. They need not be exposed outside that computer for this architecture.

## Consequences

- Proxies and adapters isolate production logic from legacy equipment-specific interfaces.
- OPC UA and MQTT can be used between local software components without requiring smart machines or external network access.
- The adapter interface for each real machine cannot be finalized until its controls, sensors and available controller interfaces are inspected.
- Running all services on one computer remains a physical single point of failure; local protocols do not provide hardware-level decentralization or availability.
