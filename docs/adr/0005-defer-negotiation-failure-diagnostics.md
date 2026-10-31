# 5. Defer negotiation failure diagnostics

Status: Accepted

## Context

When no machine commits a treatment job before its deadline, the production-process service reports that the product is not manufacturable. Negotiation can fail for different reasons, including unavailable participants, stale or inconsistent offers, and failure to reserve machine capacity. The repository does not yet have simulation or real-world test results showing which causes need dedicated diagnostics.

## Decision

For now, retain the generic "not manufacturable" outcome. Do not add a structured failure-reason protocol or a prescribed diagnostic record. Investigate why jobs fail during simulation and real-world tests, using the evidence those tests provide.

Revisit this decision if testing shows that the available evidence is insufficient to identify causes or that operators need actionable failure explanations.

## Consequences

- The current process and operator-facing outcome stay simple while the system is being tested.
- The generic outcome does not itself explain why no machine committed; investigation depends on evidence available during simulation and real-world tests.
- Test findings will inform whether structured failure reasons or additional diagnostic records are needed in a later decision.
