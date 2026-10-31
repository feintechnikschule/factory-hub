# Architecture overview

```mermaid
flowchart LR
    Machines -->|OPC UA| Edge[factory-edge-gateway]
    Edge -->|MQTT| MES[factory-mes-service]
    Edge -->|MQTT| Twin[factory-digital-twin]
    Sim[factory-simulation] -->|OPC UA / MQTT| Edge
```

## Logical architecture: decentralized machine selection

See [ADR 0002](../adr/0002-decentralized-machine-selection.md). Machine services negotiate who takes a treatment job; the production-process service only learns the outcome.

### Components

```mermaid
flowchart TD
    Operator[Operator / workplace] -->|order and start| Process[Production-process service<br/>one instance per order]

    Process -->|announce treatment job| MS1[Machine service 1]
    Process -->|announce treatment job| MS2[Machine service 2]

    MS1 <-->|negotiate: status, cost, speed| MS2

    MS1 -->|job taken / result| Process
    MS2 -->|job taken / result| Process

    Process -->|move request| Conveyor[Conveyor service]

    MS1 -->|commands and feedback| Adapter[Equipment adapters<br/>on central computer]
    MS2 -->|commands and feedback| Adapter
    Conveyor -->|commands and feedback| Adapter

    Adapter <-->|existing controls / sensors| Equipment[Machines and conveyor]
    Adapter -->|confirmed equipment state| Twin[Digital twin]
    Process -->|order progress / not manufacturable| Twin
```

### Sequence for one treatment step

```mermaid
sequenceDiagram
    actor Operator
    participant Process as Production-process service
    participant M1 as Machine service 1
    participant M2 as Machine service 2
    participant Adapter as Equipment adapter
    participant Equipment as Machine / conveyor
    participant Twin as Digital twin

    Operator->>Process: Start order for workpiece
    Process->>M1: Announce treatment job
    Process->>M2: Announce treatment job

    loop Negotiation until a machine takes the job or the limit is reached
        M1->>M2: Propose: own status, cost, speed
        M2->>M1: Propose: own status, cost, speed
        Note over M1,M2: Machine services decide who takes the job
    end

    alt A machine takes the job, here M2
        M2-->>Process: Job taken by M2
        Process-->>Twin: Update order progress
        M2->>Adapter: Command treatment
        Adapter->>Equipment: Send command
        Equipment-->>Adapter: Status and completion feedback
        Adapter-->>M2: Confirmed machine state
        Adapter-->>Twin: Confirmed machine state
        M2-->>Process: Treatment complete
        Process->>Adapter: Request carrier movement
        Adapter->>Equipment: Command conveyor
        Equipment-->>Adapter: Carrier-position feedback
        Adapter-->>Twin: Confirmed carrier position
        Adapter-->>Process: Movement confirmed
    else No machine takes the job within the limit
        Note over Process: Process service decides the limit is reached
        Process->>M1: Withdraw job announcement
        Process->>M2: Withdraw job announcement
        Process-->>Twin: Product cannot be manufactured
        Process-->>Operator: Report: not manufacturable
    end
```

### Decision ownership

| Decision | Owner |
|---|---|
| Which machine takes the job | Machine services (negotiated) |
| Whether the machine starts and performs the treatment | The machine service that took the job |
| Whether the product cannot be manufactured | Production-process service |
