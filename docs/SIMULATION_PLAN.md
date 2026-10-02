# First simulator plan

**Purpose:** Test the proposed architecture on the ground before selecting hardware or flight orbits.

This document specifies a future simulator. No implementation or passing results are claimed.

## Start with the cascade

Model one Earth trust anchor, ten first-layer relays, and forty child relays. Model the Earth command authority and Mars destination as separate logical endpoints. Report the distinction between logical roles and physical spacecraft counts.

Give each child a parent certificate relationship. Add scheduled cross-links between branches. These synthetic contacts test protocol behavior; they do not establish that a real constellation could maintain those links.

## Simulation engine

Use a deterministic discrete-event engine with a recorded random seed. Events include message creation, contact opening and closing, transmission completion, arrival, queue expiration, restart, clock update, and trust-update delivery.

Global simulation time is an oracle for evaluation only. Nodes see local clocks and received evidence; they must not access oracle time, future faults, or hidden peer status.

For each transmission, model serialization time from bundle size and bandwidth, propagation delay, queueing, and processing. A contact must have enough usable duration for the modeled transmission. Avoid treating every hop as another full Earth–Mars delay: assign per-link delays consistent with the scenario's geometry.

## Minimum node state

- Identity, signing keys, certificates, and cached trust updates.
- Clock offset, drift, and estimated uncertainty.
- Durable replay state and command results.
- Bounded queues and known contact schedules.
- Local observations and required approval policy.
- Restart behavior that preserves explicitly durable state.

Use a standard cryptographic library for signatures; do not substitute ordinary hashes for authentication. A later physics model can add ranging, Doppler, and orbit-derived predictions. The initial model must label synthetic timing evidence clearly.

## Scenarios

| Scenario | What it tests | Expected behavior |
| --- | --- | --- |
| Healthy cascade | Baseline forwarding | Valid command reaches Mars and a signed result returns. |
| Broken branch with cross-link | Route recovery | Eligible traffic uses the alternate path. |
| Partition with later contact | Store-and-forward | Unexpired queued messages resume delivery within resource limits. |
| Modified payload | Signature integrity | Destination rejects the altered command. |
| Replay and retransmission | Duplicate handling | Retries do not repeat a completed action. |
| Valid out-of-order traffic | Replay window | Unseen valid messages inside the window remain eligible. |
| Compromised relay key | Authority separation | Relay cannot sign a valid Earth command. |
| Compromised issuer | Delegation limits | Out-of-scope certificates fail; in-scope malicious children remain a modeled risk. |
| Clock drift and boundary overlap | Freshness uncertainty | Ambiguous critical commands wait rather than execute as certainly fresh. |
| Delayed revocation | Disconnected policy | Actions follow the configured trust-update age limits. |
| Conflicting valid commands | Execution policy | Destination follows explicit serialization or conflict rules. |
| Queue flooding | Resource bounds | Queue and processing limits remain enforced; measure legitimate traffic loss. |
| Restart during execution | Durable recovery | Idempotent simulated action is reconciled; report unresolved physical-action semantics. |

For each scenario, define deterministic inputs and exact assertions before running it. Do not present simulated resistance as a flight security guarantee.

## Compare three architectures

1. Direct Earth–Mars communication with a defined contact schedule.
2. A branching relay tree with no cross-links.
3. Cascade Star with branching and cross-links.

Hold message workloads, total equipment resources, fault assumptions, and contact opportunities comparable, or explicitly report the differences. Additional relays consume power, capacity, and cost; they are not free redundancy.

## Reported metrics

- Fraction of authorized messages delivered before expiration.
- End-to-end delivery and acknowledgment latency distributions.
- Expiration, rejection, and hold reasons.
- Unauthorized command executions and duplicate action count.
- Bytes transmitted per delivered payload byte.
- Queue occupancy, storage drops, and cryptographic verification workload.
- Recovery time after a branch failure.
- Exposure between compromise and effective revocation at each node.

Separate command acceptance, actual execution, and receipt of execution confirmation. Publish configuration, seed, model assumptions, and event traces sufficient to reproduce results.

## Deliverables

1. A versioned message and certificate specification.
2. A repeatable small simulator with scenario fixtures.
3. Automated checks for authorization, replay behavior, recovery, and resource bounds.
4. A comparison report, including failures and cases where relays provide no benefit.
5. A second-stage plan using ephemerides, realistic link budgets, pointing limits, occultations, and conjunction effects.

The first milestone is complete when these artifacts are reproducible and limitations are documented. Flight constellation sizes remain open until orbital and communications studies support them.
