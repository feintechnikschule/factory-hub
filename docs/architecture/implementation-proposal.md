# Base system implementation proposal

Status: Implementation sequence agreed; detailed bindings remain provisional.

## Goal

Build a small, simulated end-to-end system that demonstrates one product order moving through one equipment service and appearing in the digital twin. Use this slice to learn the concrete state and interface shapes before expanding to multiple machines, physical equipment, or coordinated order recovery.

This proposal sequences implementation; it does not finalize the detailed bindings left open by [ADR 0010](../adr/0010-interface-contract-baseline.md).

## First vertical slice

Include:

- One fixed product definition and one order fixture.
- A minimal production-process/order driver that starts the order and hands its single treatment job to one equipment proxy. Keep this as small as possible; do not build general product configuration or a full MES.
- One equipment proxy and one adapter in predefined-simulation mode. The adapter returns simulated feedback through the same local proxy boundary intended for physical feedback, and marks it as simulated.
- Durable proxy checkpoints and fail-closed persistence behavior as required by [ADR 0011](../adr/0011-persist-confirmed-proxy-state.md), including in simulation.
- One local MQTT broker. The proxy publishes observed events and state using the candidate envelope and topic pattern in ADR 0010, binding only the fields needed by this slice.
- A digital twin that subscribes to the broker and renders proxy-reported state and feedback provenance. It renders confirmed feedback, not a command or optimistic UI state.

The order driver-to-proxy handoff is an explicit temporary seam: agree a minimal contract for the slice, keep it local to the participating components, and document it as provisional rather than implying ADR 0010 has selected its production transport.

```mermaid
flowchart TD
    Fixture[One product and order fixture] --> Process[Minimal process and order driver]
    Process -->|One treatment job| Proxy[One equipment proxy]
    Proxy --> Adapter[Predefined-simulation adapter]
    Adapter -->|Simulated feedback| Proxy
    Proxy -->|Observed event and state| Broker[Local MQTT broker]
    Broker -->|Subscribe to reported state| Twin[Digital twin]
```

## Delivery sequence

1. **Agree the slice boundary.** Choose one stable demo equipment identity, one treatment service, a minimal order/job shape, and the subset of the candidate MQTT envelope needed to show the flow. Record provisional bindings in the participating component repositories; leave unrelated payload fields and broker policies open.
2. **Implement one complete simulated order.** Start the fixture order, pass the job to the proxy, let the predefined adapter produce simulated feedback, durably checkpoint confirmed transitions before reporting success, publish the resulting proxy state, and render it in the twin. Preserve the `predefinedSimulation` provenance throughout.
3. **Exercise observable behavior.** Demonstrate start, completion and an error result. Verify that a command alone does not appear as completed state, the twin updates from reported feedback, duplicate or delayed MQTT delivery does not falsely complete an order, and failed checkpoint writes prevent success reporting and further work.
4. **Expand only after the slice works.** Add a second proxy and the fail-closed negotiation of ADRs 0002 and 0004; then consider transport services, operator-driven simulation controls, the MES read path, and physical adapters after equipment inspection and interface verification.
5. **Refine timing and coordinated recovery with evidence.** Configure timing budgets for the tested deployment under ADR 0003. Refine the state and order lifecycle, checkpoint schema evolution, and coordinated order recovery while preserving the requirements of ADRs 0009 and 0011.

## Explicit non-goals for the first slice

- Physical machine control, sensor mapping, or assumptions about legacy controller signals.
- Multi-machine selection, negotiation, transport movement, or a full MES.
- Production-grade availability, host redundancy, or coordinated order recovery.
- Selecting a production storage technology before evaluating the slice's storage needs.

## Persistence requirement

[ADR 0011](../adr/0011-persist-confirmed-proxy-state.md) requires atomic durable snapshots before reporting confirmed transitions as successful. A failed write leaves the proxy unavailable for further work and requires reconciliation or reset. Storage technology and coordinated order recovery remain to be decided; neither is an exception to these requirements.

## Completion criteria

The first slice is demonstrated when one fixture order reaches a simulated completion through the proxy and adapter, confirmed transitions are durably checkpointed as required by ADR 0011, the proxy publishes the resulting feedback to the local broker, and the digital twin displays it with simulation provenance. The implementation documents provisional contracts and leaves physical behavior and coordinated order recovery explicitly unclaimed.
