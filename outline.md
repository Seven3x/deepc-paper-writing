# Paper Outline

## Working Title

`Minimal Warm-Start / Transfer for Fault-Aware DeePC under Single-Rotor Degradation`

## 1. Introduction

- Question this section answers:
- What exact fault-aware control problem is addressed for quadrotor DeePC under actuator degradation?
- Why does the old nominal/degraded scaffold fail to support a method-level claim?
- What is the minimal intervention (warm-start / transfer), and what claim boundary is enforced?

## 2. Related Problem Setup and Motivation

- Question this section answers:
- Why is single-rotor efficiency drop a controlled setting for diagnosing fault-aware DeePC behavior?
- Why are `deepc_nominal_bank` and `deepc_degraded_bank` the necessary historical baselines?
- Why is `mpc` included as a reference baseline but not the main claim target?

## 3. Method

- Question this section answers:
- What is the baseline dual-bank scaffold?
- Why do the two banks remain effectively frozen without transfer after fault onset?
- How does minimal warm-start / transfer change bank evolution while preserving the original scaffold?
- What is explicitly out of scope (fault detection, complex switching heuristics, new fault models)?

## 4. Experimental Protocol

- Question this section answers:
- What is the fixed main matrix (trajectory, scenario, onset, controller, seeds)?
- How are metrics aggregated and uncertainty reported?
- Which tables and figures are designated for main evidence versus boundary evidence?

## 5. Results

- Question this section answers:
- Does `deepc_degraded_transfer` provide average gains over the old dual-bank baselines?
- In which case/metric combinations does it improve, and where do regressions remain?
- How is the boundary to MPC stated without overclaiming?

## 6. Discussion

- Question this section answers:
- Why is transfer a necessary mechanism rather than a parameter tweak?
- Why is the current result prototype-level rather than a finalized controller outcome?
- What narrowly scoped next steps stay within the current claim boundary?

## 7. Conclusion

- Question this section answers:
- What is the strongest defensible conclusion supported by current evidence?
- What explicit non-claims (all-case dominance, MPC superiority, final-controller maturity) are maintained?
