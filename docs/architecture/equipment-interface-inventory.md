# Equipment interface inventory

This is a preliminary inventory based on the project documentation, not an inspection of the physical equipment. No controller, electrical interface, or signal has been verified. The equipment-specific adapter interfaces must remain undecided until the items below are checked on site (see [ADR 0007](../adr/0007-local-machine-proxies-and-legacy-adapters.md)).

## Known equipment and unknown interfaces

| Equipment | Known from the project description | Verify on site |
|---|---|---|
| Operator workplace | A worker places a workpiece on a carrier and presses a button to start the process. | Button function and wiring; any other operator controls or confirmations; how the adapter can observe the button action. |
| Transport system 1 | A conveyor belt moves carriers between stations. Carriers are passive. | Controller make/model and access method; available commands; carrier-presence and position sensors; home positions; how home/default and stopped/idle states can be verified. |
| Milling machine 1 | Processes a workpiece carried on the conveyor. | Controller make/model and access method; start/stop command interface; ready, busy, completion, fault, and default-state signals; how stopped/idle can be verified. |
| Transport system 2 | A second transport system exists with a second milling machine. | The same transport-system checks as system 1, including its carrier positions and home/default and stopped/idle indications. |
| Milling machine 2 | A second milling machine exists. | The same machine checks as machine 1, including its default-state and stopped/idle indications. |

The documentation does not identify whether the existing controls expose discrete I/O, a PLC interface, a vendor interface, or another connection. Do not assume a particular protocol or that either machine or transport system can report the required state.

## Inspection record to complete per asset

Record the following for each physical asset before selecting or implementing its adapter:

- Equipment identity: station/asset name, manufacturer, model, controller, and available documentation.
- Available connection: physical interface, electrical characteristics, protocol and version if applicable, and access constraints.
- Commands: available operations, required preconditions, acknowledgement behavior, and how duplicate or interrupted commands behave.
- Feedback: sensor or controller signals, meaning, units/values, update behavior, and whether each signal is direct equipment feedback.
- Recovery: how default/home state and not-running state are independently verified; if either cannot be verified, record that recovery must reset the asset (ADR 0009).
- Safety boundary: existing interlocks and emergency stops; adapter commands must not bypass them.
- Evidence and gaps: inspection date, source (for example, controller manual or observed signal), responsible contact, and unresolved questions.

Until inspection is complete, the inventory provides no basis for choosing adapter hardware or software. Simulated feedback remains distinct from physical equipment confirmation (ADR 0008).
