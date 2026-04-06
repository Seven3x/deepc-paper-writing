# Method Skeleton

## 1. Baseline scaffold

Describe the old DeePC dual-bank setup:

- `deepc_nominal_bank`
- `deepc_degraded_bank`

Explain that this scaffold separates the labels, but without continued adaptation after fault onset the two banks can remain effectively identical in their data content.

## 2. Why the scaffold is insufficient

State the failure mode:

- long pre-fault alone does not create meaningful bank divergence
- once the banks mature, the old behavior can freeze the data state
- as a result, the degraded bank does not become a truly fault-aware branch

## 3. Minimal transfer mechanism

Define the smallest change:

- start the degraded bank from the nominal mature scaffold when needed
- after fault onset, keep the degraded bank updating with post-fault samples
- keep the transfer logic simple and local, not coupled with detection, switching heuristics, or new fault models

## 4. Why this is not just tuning

Explain that this is a structural change in bank evolution, not a lambda tweak:

- it changes whether the degraded bank continues to absorb new post-fault data
- it changes whether the two banks can diverge in content after fault onset
- it is therefore a mechanism change, not a regularization sweep

## 5. What stays fixed

- same quadrotor benchmark
- same single-rotor degradation setting
- same comparison set
- same main matrix

## 6. Method claim boundary

The method claims a minimal fault-aware improvement over the old dual-bank scaffold, not a final control solution and not an MPC replacement.
