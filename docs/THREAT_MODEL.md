# Threat model
**Version:** 0.1 — design assumptions and requirements, not validated guarantees.

## Scope and protected assets
Cascade Star is a trust and routing architecture layered on DTN. Protect command authenticity, authorization, freshness, credential integrity, bounded resource use, audit history, and continued safe local operation. Availability remains limited by physical contacts and attacker capabilities.

Relays transport bundles without acquiring command authority. A compromised command source can still sign harmful commands within its permitted scope; signatures establish origin, not intent or correctness.

## Trust assumptions
- Endpoints use standard cryptographic implementations and protect private keys and durable replay state.
- Certificate and policy verification is independent of transport paths.
- Regional authorities have explicitly bounded permissions, validity, delegation depth, and key-update rights.
- A proposed 3-of-5 root requires independently protected shares and an actual threshold-signature scheme. Ordinary multisignatures are a distinct implementation option, with different formats and verification rules.
- Fewer than the signing threshold are compromised for normal root integrity. Collusion reaching that threshold is modeled as root compromise.
- Trusted local time has a bounded uncertainty estimate. Losing that bound changes allowed actions.
- Recovery anchors and emergency permissions are provisioned before the incident; they cannot be invented safely during disconnection.

## Threats and required responses
| Threat | Attacker capability | Proposed response | Residual risk / validation |
| --- | --- | --- | --- |
| Compromised relay | Drop, delay, duplicate, alter, fabricate receipts, and use its own valid relay key | End-to-end command signatures, scoped relay identity, bounded queues, alternate contacts | Can deny service on routes it controls. Cross-route observations reveal discrepancies, not necessarily their cause. |
| Compromised endpoint or stolen node key | Sign traffic within that node's permissions | Short-lived credentials, local revocation, scope limits, approval rules for critical actions | Legitimate-looking abuse remains possible until expiry or effective revocation. Local hardware compromise may defeat enforcement. |
| Compromised regional authority | Issue malicious credentials within delegated scope | Bounded scope and lifetime, independent approval for sensitive actions, threshold regional administration, signed audit and revocation | Scope limits do not protect actions already delegated to that authority. Later Earth audit cannot undo harmful execution. |
| Compromised root | Authorize malicious issuers and sign apparently valid trust changes | Threshold prevention; independently provisioned recovery quorum and recovery epoch rules | Old-root approval is insufficient when that root is compromised. If recovery authority is also compromised, no automatic trust recovery is claimed. |
| Jammed or lost link | Block a contact; potentially several contacts | Store-and-forward, eligible alternate routes, local autonomy | No delivery guarantee without a usable path before expiry and sufficient resources. |
| Replay | Resend captured valid commands or rollback old trust records | Durable message IDs, scoped sequence windows, validity checks, monotonic trust epochs | Restarts, storage loss, and key rotation need explicit state migration and recovery rules. |
| Flooding | Send invalid traffic or excessive traffic with a valid credential | Size limits before verification, admission budgets, per-authority and aggregate quotas, protected control-plane capacity | Claimed identities cannot be trusted before authentication. Verification itself consumes resources; compromised authorities can exhaust their quota. |
| Clock manipulation | Spoof timing claims, exploit drift or reset | Independent observations, bounded clock uncertainty, no peer assertion treated as ground truth | Expiry cannot be enforced confidently if local time becomes unbounded. |
| Partition and equivocation | Deliver conflicting authorized records to disconnected recipients | Versioned records, local serialization and safety policy, eventual reconciliation, conflict logs | No immediate global agreement; valid signatures alone cannot select the safe physical action. |

## Revocation and expiration limits
Revocation takes effect at a node only after an authenticated update arrives and is accepted. Certificate expiry bounds stolen-key exposure only when time is trustworthy and an attacker cannot renew the credential. A compromised issuer may renew malicious leaf credentials within its remaining authority.

Conjunction preparation cannot pre-stage knowledge of future compromises. It can pre-stage recovery policy, verification keys, delegated renewal authority, and scheduled rotations.

## Compromise recovery
Routine root rotation may be authorized by the current uncompromised root quorum. Compromise recovery instead requires a separately pinned recovery authority, monotonically increasing recovery epochs, and explicit rules disabling credentials from the superseded trust epoch. New roots cannot extend their own legitimacy merely by citing a compromised old root.

Recovery updates still need a delivery path. Mars follows pre-provisioned local restrictions while such updates are unavailable. Root continuity, recovery-quorum governance, and physical reprovisioning procedures remain to be specified and tested.

## Acceptance scenarios
1. Payload alteration cannot become an accepted Earth command.
2. Replays and retransmissions do not repeat an already completed idempotent simulated action.
3. A valid relay key cannot sign a command requiring command-authority credentials.
4. A usable alternative route recovers delivery within capacity and lifetime constraints.
5. Mars renews local leaf credentials during a simulated blackout without widening its delegated scope.
6. Stolen credentials expire or are locally revoked; record the actual exposure interval.
7. A compromised old root cannot authorize replacement of the pinned recovery authority.
8. Flooding remains within defined resource budgets; measure losses to legitimate traffic.
9. Uncertain time and conflicting valid commands lead to explicitly specified safe behavior.

## Out of scope for the first demonstration
Flight-grade tamper resistance, a complete radiation or spacecraft safety case, quantum-resistant algorithm selection, and universal resistance to coordinated physical attacks. These require separate studies. The ground experiment must report simulated assumptions, not flight guarantees.
