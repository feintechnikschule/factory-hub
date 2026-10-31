# 3. Soft-real-time timing policy

Status: Accepted

## Context

Requirement R6 calls for timing requirements for sensing, decisions and responses. The repository describes legacy equipment connected through adapters and a single central computer, but provides no measured equipment latencies or production timing requirements. A fixed numeric target would therefore be unsupported and could be unsafe to assume.

ADR 0002 currently allows negotiation to end at either a deadline or a retry limit. Without a defined timing policy, retrying could extend a job beyond its deadline, and stale equipment feedback could be treated as current.

## Decision

The system provides **soft-real-time** behavior; it does not promise hard real-time deadlines. Timing limits are explicit, positive configuration values for the deployed process and equipment, and must be measured and validated against the actual installation before commissioning. The repository does not prescribe numeric defaults.

Every operation with a timing requirement must have a configured budget, including:

- maximum feedback age when making a machine-selection decision;
- maximum negotiation duration for a treatment job;
- maximum time to confirm an equipment command; and
- maximum delay for publishing confirmed state to the digital twin.

Budgets use a monotonic clock. Retries consume the operation's original budget and never restart or extend it. For negotiation, the production-process service assigns one absolute deadline when it announces the job. A machine may not claim the job after that deadline; a separate retry limit is not used. Feedback older than the configured maximum age is not eligible for a decision.

If a configured budget expires, the operation reports a timeout or unknown state to the relevant service and digital twin; it must not present an unconfirmed command as equipment state. Safety interlocks and emergency stops remain in the existing control equipment.

Simulation tests must check behavior at and beyond configured limits. Commissioning must measure the corresponding sensing, decision and response times on the target installation and record the selected values in its deployment configuration.

## Consequences

- Timing behavior is measurable and bounded without implying that legacy equipment has hard real-time guarantees.
- Each deployment must provide and validate timing configuration before commissioning; there are intentionally no repository-wide numeric defaults.
- A timeout communicates loss of timely confirmation, not proof that equipment did not act. Recovery must use confirmed equipment feedback.
- The negotiation termination rule is unambiguous: the original deadline is authoritative, regardless of the number of retries.
