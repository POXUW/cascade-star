# Mars autonomy under Earth silence
**Version:** 0.1 — proposed regional policy.

## Authority before blackout
The Earth root delegates a bounded Mars authority. Its certificate defines allowed actions and destinations, issuance rights, maximum leaf lifetime, delegation depth, administrative approval requirements, and an authority expiry covering the planned blackout plus a justified margin.

Mars administrative keys should use a locally reachable quorum independently of Earth's root quorum. The specific membership and threshold remain governance decisions. Co-locating all shares on one compromised system defeats the independence assumption.

## Modes
| Mode | Entry condition | Permitted behavior |
| --- | --- | --- |
| Connected | Usable Earth contacts and sufficiently fresh trust updates | Normal local authority; exchange logs and updates. |
| Autonomous | Planned blackout or sustained loss of Earth contact | Operate within cached delegation and local policy; renew local leaves and issue local revocations. |
| Restricted | Authority expired, clock uncertainty exceeded, local quorum lost, or unresolved trust conflict | Predefined equipment-safe behavior and separately authorized limited emergency actions; no new general authority. |
| Recovery | Authenticated Earth or recovery updates return | Verify epochs, reconcile authority state and logs, resolve conflicts, then exit restrictions under policy. |

Loss of contact never creates new privileges or silently extends expired credentials.

## Key rotation
Each node generates a new key locally and proves possession. The Mars quorum authorizes a new leaf within the regional certificate's bounds. Scheduled overlap permits delivery of verification material before activation, but receiving nodes must enforce scope, sequence-state migration, and retirement of the old key.

Leaf lifetimes of days to weeks are candidate settings, not established requirements. Mars must be able to renew locally throughout the authorized autonomous interval. Leaf expiry alone is insufficient if a compromised regional issuer can continue renewing credentials.

## Revocation
Mars can revoke local credentials through signed, versioned regional records. These travel in high-priority bundles over every eligible path, subject to bounded replication and admission policy. Nodes retain accepted versions durably.

Earth-only credentials cannot be locally replaced unless that power was explicitly delegated. Policy may let Mars temporarily refuse their actions during a suspected compromise; refusal does not itself rewrite the Earth trust hierarchy.

Nodes disconnected from the Mars authority can still miss local revocations. Define a maximum acceptable update age by action class, and hold critical operations when it is exceeded. Availability versus stale-credential exposure is a stated policy tradeoff.

## Emergency authority
Pre-provision an independently protected Mars emergency authority with a narrow action allowlist: for example, safe shutdown, isolation of a suspect peer, or recovery of a local communications link. Specify each action's approval rule, destination, maximum duration, and audit requirement.

Emergency authority cannot install an unrestricted root, widen regional scope, disable all authentication, or renew itself indefinitely. An emergency action may remain possible after ordinary credentials expire only through an explicitly pre-provisioned policy and independent verification path. Hardware interlocks and safe local control remain mission-specific.

## Time
Track local time as an interval with bounded error; increase uncertainty during silence according to the clock model. Execute time-limited commands only when the full uncertainty interval fits their permitted window. If that becomes impossible, use restricted-mode policy rather than pretending that a timestamp is exact.

Independent clocks and ranging can constrain estimates; there is no assumed universal live clock. BP bundle age and application authorization time serve different purposes and must be mapped explicitly.

## Preparation and return
Before conjunction: distribute current trust state, recovery anchors, delegated renewal rights, scheduled rotations, contact plans, and storage budgets. Rehearse autonomous operations.

During silence: append signed, hash-linked audit records of issuance, revocation, emergency decisions, and executed commands. Protect checkpoints independently where possible. Such logs support auditing but a compromised logger can omit events; no complete-history guarantee is claimed.

On return: authenticate updates; reject rollback; compare logs and checkpoints; apply conflict and revocation policy. Regional log reconciliation is eventual. Critical actions use local approval and serialization rules, not Earth-wide synchronous consensus.

## Test
Simulate a multi-week Earth blackout, local key renewal, a stolen leaf key, interrupted regional links, drifting clocks, and loss of an administrative signer. Verify exact permissions and mode transitions, and report any interval in which compromised credentials remain accepted.
