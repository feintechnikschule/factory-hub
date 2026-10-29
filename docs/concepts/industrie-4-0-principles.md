# Industrie 4.0 principles: implications for the factory system

Summary of the discussion on the Industrie 4.0 design principles and what they mean for requirements and restrictions of this project.

## Factory context

- Worker's workplace, a conveyor belt with carriers, and a milling machine working on the workpiece from the carrier. A second transport system with a second milling machine exists.
- Workpieces and carriers are passive (no intelligence).
- The machines are **not Industry 4.0 ready**. All logic must run on **one central computer**, connected to existing controls through adapters.

## Principles applied

### Interoperability
People, machines and systems exchange information and act on it together.
- Example: the worker places a part on a carrier and presses a button; the conveyor starts; the milling machine processes the workpiece.
- Being on the same line is not enough; information must be exchanged and acted upon.

### Information transparency
A graphical simulation (digital twin) shows the factory: workplaces and machines as rectangles, the conveyor as lines, carriers as squares moving between them.
- It must show what is actually happening. Showing only issued commands shows the intent, not the real state.
- Equipment feedback (carrier positions, machine status) is needed to keep the twin in sync.

### Decentralized decisions
Example: choosing between two milling machines based on state (busy/ready) and cost vs. speed.
- A single controller that picks the machine is automated but **centralized**.
- It is decentralized when decision authority is distributed, e.g. machine services report status/cost/speed and accept or reject jobs, or the decision is taken by independent services.
- Workpieces and carriers cannot decide. A "production process" is a sequence of steps; the decision belongs to a service/controller.
- Separate hardware is not required. Logically independent services on one computer can be decentralized in decision authority, but the computer remains a physical single point of failure.

### Real-time capability
The system senses, decides and responds within the timing the production process requires.
- A twin in sync with the real process supports information transparency; real-time capability additionally requires defined timing.

### Service orientation
Capabilities and data are offered as services (machine services, conveyor service, production-process service, twin).

### Modularity
Modules (e.g. a further milling machine or transport system) can be added or rearranged with little impact on other modules.

## Requirements

| ID | Requirement |
|----|-------------|
| R1 | Workplace, conveyor and machines exchange information through defined interfaces and act on it (interoperability). |
| R2 | The digital twin shows confirmed real states (carrier positions, machine status), not only commands. |
| R3 | Equipment adapters provide feedback (sensors, status) to the services. |
| R4 | Each machine is represented by its own service reporting status (ready/busy), cost and speed, and accepting or rejecting jobs. |
| R5 | Machine selection ownership is explicit and documented per decision (central vs. distributed). |
| R6 | Timing requirements ("real time") are defined for sensing, decision and response. |
| R7 | New machines or transport systems can be added as modules without changing existing ones. |
| R8 | Services expose capabilities and data through well-defined service interfaces. |

## Restrictions and constraints

- Legacy machines: logic runs on one central computer; adapters translate to existing buttons, sensors and machine controls.
- Single point of failure: one computer hosts all services; consider this in availability and recovery planning.
- Decentralization is logical (independent services with distinct decision authority), not physical.
- Safety interlocks and emergency stops stay in the existing control equipment, never in the simulation or services.

## Next steps

1. Draw a logical component diagram (Mermaid): production-process service, machine services, conveyor service, equipment adapters, digital twin.
2. Draw a sequence for one workpiece, marking each message, command, confirmation and decision owner.
3. Record architecture decisions as ADRs in `docs/adr/` (e.g. ownership of machine selection).
