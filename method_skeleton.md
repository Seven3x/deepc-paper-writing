# Method Skeleton

## 1. Baseline Scaffold (Reference)

- Controllers in the old dual-bank scaffold:
  - `deepc_nominal_bank`
  - `deepc_degraded_bank`
- Shared structure:
  - Same DeePC backbone and same experiment protocol.
  - Degraded branch exists in name but may not evolve differently after fault onset.

## 2. Failure Mode Without Transfer: Bank Freezing

- Empirical behavior to state:
  - Extending pre-fault phase alone does not produce reliable nominal/degraded divergence.
  - After maturity, degraded bank can stay effectively frozen with nominal-like data content.
- Method implication:
  - Dual-bank labels alone are insufficient for fault-aware behavior.
  - A mechanism for post-fault degraded-bank evolution is required.

## 3. Minimal Warm-Start / Transfer Mechanism

- Mechanism definition:
  - At degraded bank first activation, initialize from mature nominal bank (`warm-start`).
  - After fault onset, keep degraded bank adapting using post-fault sliding-window samples (`transfer/adapt`).
- Implementation shape:
  - Minimal local addition to bank update flow.
  - No expansion into detection, mode-estimation, or complex switching policies.

## 4. Why This Is Method Change, Not Hyperparameter Tuning

- What changes:
  - Whether degraded bank receives and internalizes post-fault samples.
  - Whether nominal and degraded banks can diverge in data-state after fault onset.
- What does not define the change:
  - Not a weight/lambda sweep.
  - Not a solver-only tweak.
- Therefore:
  - This is a structural bank-evolution mechanism change.

## 5. What Stays Fixed (To Isolate Mechanism Effect)

- same quadrotor benchmark
- same single-rotor degradation setting
- same comparison set
- same main matrix

## 6. Implementation and Claim Boundary

- Explicit exclusions:
  - No fault detection module.
  - No complex bank switching heuristics.
  - No broadened multi-fault framework in this paper.
- Claim scope:
  - Minimal fault-aware improvement over old dual-bank scaffold.
  - Prototype-level method contribution, not final controller and not an `mpc` replacement claim.
