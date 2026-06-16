# CROSS-POLLINATION.md — ternary-fleet

> **Conservation Law Connection:** γ + η = C — this IS the fleet

## Role in the Conservation Law

`ternary-fleet` is the umbrella workspace containing the operational fleet crates:
activation, bite, checkpoint, conv, distill, em, fuse, hmm, and more. Each sub-crate
is a fleet capability that contributes to either γ (productive computation) or
η (coordination overhead).

The fleet architecture directly implements the conservation law in code:
- **γ components:** ternary-conv (computation), ternary-fuse (integration), ternary-em (inference)
- **η components:** ternary-checkpoint (state management), ternary-activation (signaling)
- **C constant:** the total budget, enforced by conservation-action CI checks

## delta-clt Verification Results

The delta-clt dependency graph simulation (Section 5) models exactly this architecture:
nodes = sub-crates, edges = data flow, γ = payload throughput, η = routing + health cost.

Results at n=50 nodes: γ/C ≈ 97%, η/C ≈ 3% — the fleet architecture is efficient
at scale. At n=10: γ/C ≈ 91%, η/C ≈ 9% — small fleets pay proportionally more overhead.

The live experiment (6 agents) showed 18.6% drift, meaning the fleet at small scale
is still finding its equilibrium. The prediction holds that at ≥50 agents, drift < 1%.

## Cross-Repo Connections

### → ternary-fleet-integration
`ternary-fleet-integration` provides the integration tests that verify sub-crates
work together. It tests the γ side: do components produce correct results when combined?
The η side (coordination overhead) is tested by delta-clt simulations.

**Shared:** Both are about the fleet working as a whole. Integration tests verify γ
quality; delta-clt verifies η bounds.
**Different:** `ternary-fleet` is the implementation; `integration` is the test harness.

### → ternary-fleet-packing
`ternary-fleet-packing` optimizes how fleet components are packed into binary
deployments. Packing density affects η — poorly packed fleets have higher routing
overhead. The conservation law predicts optimal packing density of 1 − δ(n).

**Shared:** Both optimize fleet-level properties.
**Different:** `fleet` defines capabilities; `packing` optimizes their deployment footprint.

### → superinstance-core
`superinstance-core` provides the ECS entity management that the fleet runs on.
Each fleet sub-crate registers components with the ECS. The ECS is the substrate
on which γ and η flow.

**Shared:** Both are infrastructure for the fleet.
**Different:** `fleet` is the application layer; `core` is the entity system.

## Fleet Position

```
┌──────────────────────────────────────────────────────┐
│  ternary-fleet — THE FLEET ITSELF                     │
│                                                       │
│  ┌─────────────┐  ┌──────────────┐  ┌─────────────┐  │
│  │ ternary-conv│  │ ternary-fuse │  │ ternary-em  │  │
│  │   (γ)       │  │   (γ)        │  │   (γ)       │  │
│  └─────────────┘  └──────────────┘  └─────────────┘  │
│  ┌─────────────────┐  ┌───────────────────────────┐  │
│  │ ternary-checkpt │  │ ternary-activation        │  │
│  │   (η)           │  │   (η)                     │  │
│  └─────────────────┘  └───────────────────────────┘  │
│                                                       │
│  Tested by: ternary-fleet-integration (γ)            │
│  Verified by: delta-clt (η bounds)                   │
│  Deployed via: ternary-fleet-packing                 │
│  Runs on: superinstance-core                          │
└──────────────────────────────────────────────────────┘
```

