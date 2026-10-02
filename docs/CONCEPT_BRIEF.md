# Cascade Star: Concept Brief
**A trust and routing architecture layered on DTN.**

## Problem
Earth–Mars messages face roughly 3–22 minutes of one-way light time and intermittent contacts. Solar conjunction recurs about every 26 months; a roughly two-week command blackout is a planning scenario whose duration varies by mission. Mars cannot depend on a live Earth authorization response.

DTN transports bundles through delay and disruption. Cascade Star proposes how delegated authority, local verification, disconnected key administration, and trust-aware routing fit around that transport.

## Principles
1. End-to-end authentication: relays cannot forge a command-authority signature merely by forwarding it.
2. Bounded store-and-forward: protect storage and control-plane resources.
3. Path diversity over an independent contact graph: compare observations without claiming every discrepancy proves censorship.
4. Scoped Mars autonomy: maintain local credentials, revocations, and narrowly defined emergency actions under pre-provisioned policy.
5. Eventual reconciliation of distributed administrative records, with local approval and serialization for critical operations. Light-minute delays make interactive global agreement costly; they do not make all agreement protocols impossible.

## Trust structure
A proposed threshold root, such as 3-of-5 independently governed signers, delegates bounded regional authority. Mars renews short-lived local leaves without Earth contact and distributes signed trust updates over DTN. Expiration limits exposure only with trustworthy time and uncompromised renewal authority.

Routine root rotation and compromised-root recovery are different procedures. Recovery uses an independently pinned authority and rollback-resistant epochs; a compromised old root cannot vouch for its own replacement.

## Minimum demonstration and growth
Use Earth, one candidate Sun–Earth L4/L5 relay, two Mars orbiters, and one surface node. Test candidate contacts and local autonomy separately; no conjunction bypass is assumed.

Growth is triggered by demonstrated traffic, coverage, new-outpost, or redundancy needs. The 1 → 10 → 40 pattern is an illustration of delegation expansion, not physical shape or a deployment target.

## Next steps
1. Review the [threat model](THREAT_MODEL.md).
2. Specify certificate encoding, scope, endpoint binding, and verification rules.
3. Specify [Mars autonomy](MARS_AUTONOMY.md) and the [minimum network](MINIMUM_NETWORK.md).
4. Evaluate an existing DTN implementation such as ION or DTNME; neither is selected or integrated yet.
5. Simulate compromised relays, replay, stolen keys during blackout, and independent root recovery.

Governance, emergency privileges, traffic quotas, clock tolerances, and measurable expansion thresholds remain open decisions.
