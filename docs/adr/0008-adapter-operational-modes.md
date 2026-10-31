# 8. Equipment adapter operational modes

Status: Accepted

## Context

Machine proxies and their equipment adapters are part of the simulation setup. An adapter can operate with its physical machine connected or without a physical connection. In the disconnected mode, the adapter can either derive sensor/control feedback from a predefined simulated situation using the commands it receives, or connect to a digital-twin interface where a simulation operator generates that feedback using buttons and checkboxes.

The proxy should receive feedback through its adapter in each mode. Commands alone are intent and must not be presented as confirmed equipment state, consistent with the information-transparency requirement and ADR 0003.

## Decision

Each adapter has two top-level operational modes:

1. **Physical machine connected:** forward commands to the physical machine and return its sensor/control feedback to the machine proxy.
2. **Physical machine not connected (simulation):** do not send commands to the physical machine. Select one simulation feedback source for the adapter:
   - **Predefined situation:** use received commands to look up or advance the configured simulated situation and return its corresponding sensor/control feedback.
   - **Operator-driven digital twin:** connect to the digital twin's simulation controls. Buttons and checkboxes operated by the simulation operator generate sensor/control feedback, which the adapter returns to the proxy.

In all modes, the proxy receives state feedback from its adapter. The digital twin may display state received from the system; in operator-driven simulation it additionally provides explicit simulated inputs. Operator-generated feedback is simulation input, not physical sensor confirmation, and must remain distinguishable from feedback from a connected physical machine.

Only one mode and one feedback source are active for an adapter at a time. The concrete mode-selection mechanism, feedback schema, scenario lookup rules, and the mapping between digital-twin controls and feedback values remain implementation details for the simulation and component repositories.

## Consequences

- The same proxy and adapter boundary can be exercised with physical equipment, deterministic predefined scenarios, or operator-driven simulation.
- Physical commands are isolated from simulated operation when the adapter is disconnected from the physical machine.
- Simulation can test command/feedback behavior without claiming that simulated feedback is a physical observation.
- Mode changes and feedback provenance need to be visible to simulation operators and preserved by implementations; this ADR does not prescribe a UI or telemetry schema.
