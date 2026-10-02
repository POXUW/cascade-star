# First simulator plan

**Purpose:** Test Cascade Star's trust and routing policies over DTN on the ground before selecting hardware or flight orbits.

This document specifies a future simulator. No implementation or passing results are claimed.

## Threat-driven starting point

Use the [threat model](THREAT_MODEL.md) to set assertions first. Begin with the [five-endpoint minimum network](MINIMUM_NETWORK.md); evaluate ION or DTNME integration before choosing a stack. Model scoped [Mars autonomy](MARS_AUTONOMY.md), including local renewal and revocation through a multi-week Earth blackout. Test root recovery with a compromised old root and an independent pinned recovery authority. No DTN stack has been selected or integrated.

## Later delegation-scale fixture

After the minimum demonstration, a separate scaling experiment may model an Earth trust anchor, ten first-layer delegated identities, and forty child identities. Model the Earth command authority and Mars destination as separate logical endpoints. These counts describe logical delegation, not spacecraft or link requirements.

Build the certificate hierarchy and time-varying DTN contact graph independently. A node may forward through a peer outside its certificate branch. Include multiple non-tree contact layouts and a case where a certificate parent is unreachable but a valid route still exists.

Synthetic contacts test policy behavior; they do not establish that a real constellation could maintain those links. A graph simulator is not a BPv7 implementation. Either integrate a chosen DTN stack or explicitly document which BPv7/BPSec semantics the model approximates and which are unimplemented.

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
| Broken physical route | Route recovery | Eligible traffic uses an alternate DTN path, independent of certificate ancestry. |
| Unreachable certificate parent | Trust/contact separation | A cached valid certificate chain permits eligible traffic over an unrelated route. |
| Reachable unauthorized peer | Route eligibility | Physical reachability alone does not permit a prohibited route or action. |
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

## Compare policies on the same DTN network

1. Baseline DTN routing with defined end-to-end authentication and resource limits.
2. DTN routing with Cascade Star's identity and authorization constraints.
3. The same constraints plus optional physical consistency evidence.

Hold topology, contact schedule, workloads, cryptographic baseline, equipment resources, and fault assumptions constant to isolate policy effects. Report availability costs from stricter policy as well as security benefits.

Separately compare physical mission layouts, including direct Earth–Mars links and relay configurations. Label those as topology studies; extra connectivity is not evidence of a benefit from the trust architecture itself.

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
2. A repeatable DTN model or integrated stack with independent trust and contact graphs, scenario fixtures, and a documented standards-coverage boundary.
3. Automated checks for authorization, replay behavior, recovery, and resource bounds.
4. A comparison report, including failures and cases where relays provide no benefit.
5. A second-stage plan using ephemerides, realistic link budgets, pointing limits, occultations, and conjunction effects.

The first milestone is complete when these artifacts are reproducible and limitations are documented. Flight constellation sizes remain open until orbital and communications studies support them.
