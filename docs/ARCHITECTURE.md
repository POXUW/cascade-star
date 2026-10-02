# Cascade Star architecture draft

**Version:** 0.1 — proposed behavior for discussion and simulation.

## The central rule

Trust branches outward, but authority remains bounded. A relay may transport a command without gaining permission to create that command, change it, or execute it.

The cascade has two related structures:

- A **trust hierarchy** records who authorized each node and what it may do.
- A **contact graph** records which nodes can communicate, when, and at what capacity.

A child keeps its certified identity even when it routes through a different branch. Losing contact with a parent does not automatically revoke the child.

## Node roles

| Role | Responsibility |
| --- | --- |
| Earth root | Authorize issuers and distribute signed trust policy; remain offline for routine traffic where practical. |
| Command authority | Sign commands for specific destinations and actions. Its authority is separate from relay certification. |
| Delegated issuer | Certify children within allowed scope, depth, and validity limits. |
| Relay | Check envelopes, apply forwarding policy, store bundles, and report receipt. |
| Destination | Verify command authority and execution conditions; maintain durable replay protection. |

A physical spacecraft may perform several roles. Its certificates must make those permissions explicit. The root private key is never distributed to relays.

## Growing from 1 to 10 to 40

The numbers describe expansion, not a security threshold. An initial issuer could authorize ten relays, and those relays could support forty children across their branches.

Each child joins by generating a key pair, proving possession of its private key, and receiving a signed certificate. Enrollment requires an authorized provisioning process; discovering a radio signal is insufficient.

A certificate includes a node identifier, public key, issuer, allowed roles, permitted destinations or actions, validity interval, delegation limits, and policy version. A child cannot grant permissions beyond its issuer's delegated scope.

Root updates, revocations, and policy changes use signed, monotonically versioned records. Nodes retain the highest accepted versions in durable storage to resist rollback after a restart.

## Message envelope

Before implementation, select one unambiguous canonical encoding and domain-separated signing format. The following is a field proposal, not a wire protocol:

| Field | Purpose |
| --- | --- |
| protocol_version / network_id | Separate versions and networks. |
| message_id | Unique identifier, covered by the signature. |
| message_type | Distinguish commands, telemetry, trust updates, and receipts. |
| origin_id / signing_key_id | Identify the originating authority. |
| destination_id | Bind the message to its intended recipient. |
| stream_id / sequence | Order traffic within a defined origin, key, destination, and stream scope. |
| issued_at / expires_at | Define validity in the agreed timescale. |
| policy_version | Identify the authorization rules expected by the sender. |
| payload / payload_digest | Bind the complete payload to the signature. |
| signature | Authenticate all immutable envelope fields and the payload binding. |

Forwarding metadata lives outside the signed original: next-hop information, local receipt times, and optional signed hop receipts. It cannot override the destination, payload, or authorization.

Sequence numbers are not shared globally across unrelated traffic. Receivers maintain a bounded replay window so valid messages arriving out of order are not discarded merely because a higher sequence arrived first. Certificate rotation must define the replay-state transition.

## A command from Earth to Mars

```mermaid
sequenceDiagram
    participant E as Earth command authority
    participant A as Relay A
    participant B as Relay B
    participant M as Mars destination
    E->>A: Signed command and certificate references
    A->>A: Verify; store until contact
    A->>B: Forward original signed envelope
    B->>B: Verify; store until contact
    B->>M: Forward original signed envelope
    M->>M: Check authority, freshness, replay state, execution policy
    M-->>B: Signed execution result
    B-->>A: Forward result when contact permits
    A-->>E: Forward result when contact permits
```

A relay receipt means a relay accepted a bundle, not that Mars executed it. Execution confirmation must come from the destination. Lost acknowledgments may trigger retransmission of the same message identifier, not a new command identifier.

## The local court

Each node evaluates messages using locally available evidence and a signed policy.

| Outcome | Example trigger | Behavior |
| --- | --- | --- |
| Reject | Invalid signature, unauthorized action, known revocation, or proven expiration | Do not execute; record a bounded diagnostic. |
| Hold | Missing certificate, unresolved policy conflict, uncertain validity, or required approvals unavailable | Store within resource limits; seek missing evidence. |
| Forward | Envelope is valid and forwarding is allowed | Queue for an eligible contact; this does not authorize execution. |
| Execute | Destination checks and action-specific approval policy pass | Apply durable duplicate protection and record the result. |

Forwarding and execution are separate state machines. A message may be forwarded and still require further evidence at its destination. Missing connectivity is not evidence that a node is malicious.

For physical actions, durable logging alone cannot guarantee exactly-once effects after a crash. The executor needs idempotent actions or a device-specific recovery procedure that reconciles recorded intent with actual equipment state.

## Time and physical evidence

Track time as an estimate with uncertainty rather than an exact reading. If a node's estimated current time lies in interval [earliest, latest], execution inside a validity window requires the whole interval to fit that window under the selected policy. A boundary overlap produces a hold rather than a claim of certainty.

Relays use separate storage and forwarding limits so uncertain execution eligibility does not force indefinite storage. The system must define behavior after clock reset or loss of synchronization.

Two-way ranging, Doppler observations, orbit predictions, and relativistic clock corrections provide consistency evidence with explicit error bounds. A timestamp supplied by a potentially compromised peer is a claim, not an independent measurement. Multiple observations from the same source retain that common dependency.

These checks cannot reliably identify every malicious relay: a compromised relay may report physically plausible data. Physical inconsistency can justify investigation or route avoidance without proving compromise.

## Independent approvals

Ordinary transport does not require every node to vote. Some critical actions may require signatures from several designated authorities on the same command digest, destination, policy version, and validity interval.

An initial simulator can explore a 2-of-3 approval policy from independently provisioned authorities. This is an authorization rule, not a Byzantine consensus protocol. It does not establish globally consistent ordering or prevent two authorities from approving conflicting commands.

Define conflict handling and execution serialization separately. Full Byzantine agreement, if needed, requires a specific protocol and justified assumptions about membership, faults, and network availability. Descendants of one relay do not count as independent observations merely because they have different keys.

## Routing and branch recovery

Maintain a contact plan describing anticipated link windows, capacity, propagation delay, and uncertainty. Choose routes subject to message expiration, storage availability, traffic priority, and authorization constraints. Limit replication to avoid exhausting bandwidth and memory.

When a branch stops responding:

1. Mark its contact as unavailable; do not infer malicious intent from silence.
2. Try eligible cross-links and alternate contacts.
3. Preserve the original message identifier and signature during retries.
4. Store until an allowed deadline if no viable route exists.
5. Report failure or expiration when a return route is available.

Restrict queue sizes, per-origin traffic, and processing costs. Check framing and size limits before expensive cryptographic operations. Prevent forwarding loops with duplicate tracking and bounded hop policy, while retaining the immutable original envelope.

## Revocation during disconnection

Every node records the newest authenticated trust update it knows. It cannot know about a newer revocation until information reaches it.

Policies therefore specify an allowed trust-update age by action class. Critical execution may pause when this age is exceeded; lower-risk forwarding may continue under separate rules. Certificate lifetimes and trust-update freshness depend on clock uncertainty and mission contact schedules.

A compromised issuer can authorize malicious children within its scope before revocation. Scope limits, independently authorized approvals, and route diversity reduce exposure; they do not eliminate it. Recovery from root compromise must be designed before deployment, including a separately provisioned recovery authority and rollback-resistant transition rules.

## Relation to existing work

Evaluate existing Delay-Tolerant Networking mechanisms before inventing transport: Bundle Protocol Version 7 (RFC 9171) and Bundle Protocol Security (RFC 9172) are relevant starting points. Cascade Star's proposed contribution is how delegated trust, local verification courts, physical consistency checks, and branch routing fit together. This draft does not claim those individual mechanisms are new.

## Decisions still required

Cryptographic algorithms and encoding; hardware key protection; authority independence; certificate and replay-state lifecycle; resource budgets; root recovery; physical measurement error models; orbit placement; and action-specific rules for uncertainty, partitions, and conflicting commands.
