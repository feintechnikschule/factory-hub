# Architecture overview

```mermaid
flowchart LR
    Machines -->|OPC UA| Edge[factory-edge-gateway]
    Edge -->|MQTT| MES[factory-mes-service]
    Edge -->|MQTT| Twin[factory-digital-twin]
    Sim[factory-simulation] -->|OPC UA / MQTT| Edge
```
