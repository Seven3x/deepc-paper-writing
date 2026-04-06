# Introduction Skeleton

## 1. Problem Setting: Data-Driven Quadrotor Control Under Actuator Degradation

- Context:
  - Quadrotor outer-loop tracking with DeePC in a model-light, data-driven setting.
  - Fault type fixed to `single-rotor efficiency degradation` in simulation.
- Core difficulty:
  - Nominal tracking is easier than post-fault tracking.
  - After fault onset, controller must adapt to changed plant behavior, not just keep nominal behavior.

## 2. Why the Old Nominal/Degraded Scaffold Is Insufficient

- Old scaffold controllers:
  - `deepc_nominal_bank`
  - `deepc_degraded_bank`
- Observed issue:
  - Without transfer, the degraded bank can remain effectively tied to the same data state as nominal.
  - Bank names differ, but adaptive content does not meaningfully diverge.

## 3. Why This Is a Bank-Evolution Mechanism Problem (Not Just Tuning)

- Key diagnosis:
  - Long pre-fault phase alone does not force useful nominal/degraded divergence.
  - The bottleneck is whether degraded bank keeps evolving with post-fault data.
- Consequence:
  - If evolution is frozen, dual-bank scaffold cannot support a defensible fault-aware method claim.
- Clarification:
  - This is not primarily a regularization/weight/penalty retuning story.
  - It is about data-state evolution of the degraded bank.

## 4. What We Change: Minimal Warm-Start / Transfer

- Minimal mechanism:
  - Warm-start degraded bank from mature nominal bank at first use.
  - Continue degraded bank updates with post-fault sliding-window samples after fault onset.
- Scope control:
  - Keep the dual-bank scaffold and benchmark protocol otherwise fixed.
  - No added fault detection module, no large reconfiguration framework.

## 5. What the Current Evidence Supports

- In the fixed main matrix (`step/figure8`, `mild/severe`, onset `20.0s`, fixed seeds):
  - `deepc_degraded_transfer` shows stable average improvement versus old dual-bank baselines.
  - Gains are not uniform across all case-metric pairs.

## 6. Claim Boundary for This Paper

- Claimable:
  - A fault-aware DeePC method improvement at prototype level.
  - Minimal transfer is a necessary mechanism to make degraded branch usable.
- Not claimable:
  - Not all-case/all-metric dominance over old banks.
  - Not superior to `mpc`.
  - Not a finalized mature fault-tolerant controller.
