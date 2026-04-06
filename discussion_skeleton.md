# Discussion Skeleton

## 1. Why transfer is necessary

- Re-state the mechanism-level finding: without transfer, nominal/degraded banks can remain frozen in the same data state even with long pre-fault operation.
- Explain consequence: the old dual-bank scaffold has labels but weak functional separation.
- Explain what transfer changes: degraded bank receives post-fault adaptation updates, enabling actual bank evolution after onset.

## 2. Why this is not yet a mature final solution

- Performance is improved on average but not uniformly across all case-metric pairs.
- Mild cases expose mixed behavior and instability of gains.
- MPC remains the stronger reference baseline, so current method should be framed as prototype-level.
- Therefore, claim scope is method improvement with bounded evidence, not finalized fault-tolerant control.

## 3. Limitations (explicit and concrete)

- Only one fault family is tested: single-rotor efficiency drop.
- Only one onset timing is used: `20.0s`.
- Only two trajectories are used: `step` and `figure8`.
- No integrated fault detection or advanced switching policy is included.

## 4. Focused next steps (non-generic, post-writeup scope)

- Stabilize mild-case regressions (`step + mild` final position; `figure8 + mild` position/yaw) without changing the problem scope.
- Improve transfer update policy robustness under the same fixed matrix before any scenario expansion.
- After writeup completion, evaluate whether a small targeted robustness check is needed; avoid reopening broad sweeps.

## 5. Discussion closing sentence template

- "Minimal warm-start / transfer is a necessary mechanism to make this DeePC dual-bank scaffold fault-aware, but current evidence supports a prototype-level claim rather than a mature controller claim."
