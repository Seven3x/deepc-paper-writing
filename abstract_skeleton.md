# Abstract Skeleton

1. Background:
We study data-driven fault-aware DeePC control for a lightweight quadrotor under single-rotor efficiency degradation.

2. Old scaffold gap:
The nominal/degraded dual-bank scaffold is insufficient by itself because, without continued post-fault adaptation, both banks can remain effectively frozen in the same data state.

3. Minimal transfer mechanism:
We add a minimal warm-start / transfer mechanism so the degraded bank is bootstrapped from the mature nominal bank and continues adapting with post-fault samples while the rest of the scaffold remains unchanged.

4. Main-matrix result:
On the fixed main matrix (`step` and `figure8`; `mild` and `severe`; onset `20.0s`; seeds `41-50`), this mechanism yields stable average improvement over the old nominal/degraded banks, with explicit boundary-case regressions remaining.

5. Claim boundary:
We do not claim superiority to MPC or all-case/all-metric dominance; the method is positioned as a prototype-level but defensible fault-aware DeePC improvement.
