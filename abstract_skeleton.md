# Abstract Skeleton

We study fault-aware DeePC for a lightweight quadrotor under single-rotor efficiency degradation.

The nominal/degraded dual-bank scaffold is not sufficient on its own: when the degraded bank does not continue adapting after fault onset, the two banks can remain effectively frozen in the same data state and fail to produce a meaningful method-level distinction.

To address this, we introduce a minimal warm-start / transfer mechanism that keeps the degraded bank adapting after fault onset while preserving the original dual-bank structure.

In a frozen single-rotor degradation benchmark with `step` and `figure8` trajectories, `mild` and `severe` fault settings, and fixed seeds, the proposed transfer mechanism produces a clear average improvement over the old nominal/degraded banks.

The result does not claim dominance over MPC and does not claim a fully finalized controller. Instead, it shows that minimal transfer is the missing mechanism needed to turn the dual-bank DeePC scaffold into a fault-aware prototype with consistent gains under the tested degradation setting.
