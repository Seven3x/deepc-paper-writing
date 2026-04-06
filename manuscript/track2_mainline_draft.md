# Track 2 Mainline Draft

## Title

Minimal Warm-Start / Transfer for Fault-Aware DeePC under Single-Rotor Degradation

## Abstract

We study data-driven fault-aware DeePC control for a lightweight quadrotor under single-rotor efficiency degradation. Our starting point is a nominal/degraded dual-bank DeePC scaffold. However, the scaffold alone is insufficient for a method-level fault-aware claim: without continued post-fault adaptation, the degraded bank can remain effectively frozen in the same data state as the nominal bank. We therefore add a minimal warm-start / transfer mechanism in which the degraded bank is bootstrapped from a mature nominal bank and continues updating with post-fault samples after fault onset, while the rest of the scaffold remains unchanged. On a fixed benchmark covering `step` and `figure8` trajectories, `mild` and `severe` degradation, onset `20.0s`, and 10 seeds, this mechanism yields stable average improvement over the old nominal/degraded banks, while still leaving explicit boundary-case regressions in mild settings. We do not claim superiority to MPC or all-case/all-metric dominance. Instead, the result is positioned as a prototype-level but defensible fault-aware DeePC improvement.

## 1. Introduction

Data-driven control is attractive for quadrotor tracking because it can reduce modeling burden while preserving a direct connection between measured behavior and control synthesis. In this project, the target setting is outer-loop position tracking with DeePC under actuator degradation. The difficulty is not nominal tracking alone, but retaining useful closed-loop behavior after the plant changes because of a fault. A controller that remains tied to nominal data after fault onset may continue to function, but it does not yet support a convincing fault-aware interpretation.

Track 2 centers on a nominal/degraded dual-bank DeePC scaffold. This scaffold is a reasonable starting point because it separates nominal and degraded branches conceptually. However, our recent results show that the scaffold is not sufficient by itself. When the degraded bank does not continue adapting after fault onset, the nominal and degraded banks can remain effectively frozen in the same data state. In that regime, the controller has two bank labels but not two meaningfully different adaptive states.

This diagnosis matters because it changes how the method should be improved. The main bottleneck is not best understood as parameter tuning. Extending the pre-fault phase alone does not force useful divergence between the banks. The key issue is whether the degraded branch continues to evolve with post-fault samples. If it does not, the dual-bank scaffold does not become genuinely fault-aware in practice.

To address this issue, we introduce a minimal warm-start / transfer mechanism. The degraded bank is bootstrapped from the mature nominal bank when first needed, and then continues to absorb post-fault data through a sliding-window update process after fault onset. This keeps the overall scaffold simple: we do not add a fault detection module, a large reconfiguration framework, or a more elaborate switching policy. The change is intentionally narrow and local to bank evolution.

The resulting evidence supports a conservative paper position. In the fixed single-rotor degradation matrix used for the paper, `deepc_degraded_transfer` provides stable average gains over the old `deepc_nominal_bank` and `deepc_degraded_bank` baselines. At the same time, the improvement is not uniform across all case-metric pairs, and the method remains weaker than MPC. The contribution of this paper is therefore not a final controller claim. It is a bounded method improvement showing that minimal transfer is the missing mechanism that turns a frozen dual-bank DeePC scaffold into a usable fault-aware prototype.

## 2. Problem Setup and Motivation

The benchmark in this paper is intentionally narrow. We focus on a lightweight quadrotor simulation subject to single-rotor efficiency degradation. This fault model is useful because it creates a controlled departure from nominal dynamics without expanding into a large family of unrelated failures. The benchmark is restricted to two reference trajectories, `step` and `figure8`, and two degradation severities, `mild_single_rotor_drop` and `severe_single_rotor_drop`, with a fixed onset time of `20.0s`. This restriction is deliberate: the paper aims to establish the minimum claim that is currently supported, not to generalize beyond the available evidence.

The old dual-bank scaffold provides two historical baselines. `deepc_nominal_bank` represents the nominal bank path, and `deepc_degraded_bank` represents the degraded bank path without the new transfer mechanism. These baselines remain necessary because the main question is not whether DeePC can beat every external controller, but whether the fault-aware branch becomes meaningfully different and more useful once transfer is added. MPC is still included in the experiments, but only as a stronger reference baseline that anchors the current maturity level of the DeePC prototype.

## 3. Method

### 3.1 Baseline dual-bank scaffold

The baseline scaffold contains two controller variants: `deepc_nominal_bank` and `deepc_degraded_bank`. Both share the same DeePC backbone and operate under the same experimental protocol. The intended interpretation is that one bank reflects nominal operation while the other reflects degraded operation. In practice, however, that interpretation only becomes meaningful if the degraded bank actually evolves differently after fault onset.

### 3.2 Failure mode without transfer

The main empirical failure mode is bank freezing. Even after a long pre-fault phase, the degraded bank does not automatically become distinct from the nominal bank. Once both banks have matured under similar nominal data, the degraded branch can remain effectively locked to nominal-like content. In that case, the old scaffold does not provide a strong basis for a fault-aware method claim. The issue is structural: the degraded bank lacks a mechanism for sustained post-fault evolution.

### 3.3 Minimal warm-start / transfer mechanism

The method change in this paper is intentionally minimal. When the degraded bank is first activated, it is initialized from the already mature nominal bank. After fault onset, the degraded bank continues to update using post-fault sliding-window samples. The implementation uses a simple periodic adaptation flow rather than reconstructing the bank at every step. This gives the degraded branch a practical way to keep incorporating post-fault information while preserving the rest of the scaffold.

### 3.4 Why this is a method change rather than tuning

This modification should not be described as ordinary tuning. The key difference is not a changed penalty weight or a solver setting. The key difference is whether the degraded bank internalizes post-fault data and therefore becomes capable of diverging from the nominal bank after onset. Because the change acts on bank evolution itself, it is better understood as a method-level mechanism change.

### 3.5 Scope boundary

The method boundary is explicit. This paper does not introduce a fault detection module. It does not add complex switching heuristics, broaden the fault family, or claim a full reconfiguration framework. The contribution is narrower: minimal warm-start / transfer makes the degraded branch practically usable within the current dual-bank DeePC scaffold.

## 4. Experimental Protocol

The paper uses one fixed main matrix. The trajectories are `step` and `figure8`. The fault scenarios are `mild_single_rotor_drop` and `severe_single_rotor_drop`. The fault onset is fixed at `20.0s`. The controller set is `deepc_nominal_bank`, `deepc_degraded_bank`, `deepc_degraded_transfer`, and `mpc`. The seed set is `41` through `50`, yielding 10 seeds and 160 total runs.

The primary metrics are position RMSE, final position error norm, and yaw RMSE. Success rate is included as a reliability summary. For each case, we aggregate results across seeds using mean and standard deviation, and we use 95% confidence intervals or equivalent error bars to visualize seed-level uncertainty. To isolate the effect of transfer, we also report deltas of `deepc_degraded_transfer` relative to the old dual-bank baseline. In the strengthened paper matrix, `deepc_nominal_bank` and `deepc_degraded_bank` are numerically identical, so the deltas are the same for both historical baselines.

The figure and table organization follows the paper claim boundary. The main text should contain the aggregated main table, the CI bars, and a representative advantage case. The appendix should contain the transfer delta table, the boundary case, the seed-level summary, the delta heatmap, and the average old-versus-transfer overview.

## 5. Results

### 5.1 Main result: transfer breaks the frozen pattern

The first result to state is structural: in the strengthened main matrix, `deepc_nominal_bank` and `deepc_degraded_bank` are numerically identical across the reported aggregate metrics. This confirms that the old scaffold still behaves like a frozen dual-bank setup under the paper protocol. Against that background, `deepc_degraded_transfer` introduces non-trivial separation from the old banks. The method is therefore doing more than relabeling a branch; it changes the adaptive behavior of the degraded path.

### 5.2 Average gain relative to the old dual-bank baseline

Relative to the old banks, the cross-case average deltas of `deepc_degraded_transfer` are `-10.21` in position RMSE, `-99.12` in final position error, and `-649.91` in yaw RMSE. This is the core positive result of the paper. The gain is not a single-seed accident, and it is visible across the fixed matrix as an average effect.

At the case level, the strongest representative example is `step + severe_single_rotor_drop`, where transfer improves position RMSE, final position error, and yaw RMSE relative to the old banks. This is the cleanest main-text example of the intended fault-aware benefit.

### 5.3 Boundary cases and explicit regressions

The paper should keep the mixed cases visible rather than hiding them. In `step + mild_single_rotor_drop`, transfer improves position and yaw but regresses in final position error by `+83.80` relative to the old banks. In `figure8 + mild_single_rotor_drop`, transfer improves final position error but regresses in position RMSE by `+3.96` and yaw RMSE by `+46.78`. These cases matter because they define the maturity boundary of the method. The evidence supports average improvement, not uniform dominance.

### 5.4 MPC as reference only

MPC remains a stronger reference baseline in this benchmark, especially on the position and final-position metrics that matter most for the current task. That fact should be stated directly. However, the comparison to MPC is not the main logic of the paper. Its role is to prevent overclaiming and to make clear that the current DeePC result is still a prototype-level method improvement.

## 6. Discussion

The main discussion point is that transfer is a necessary mechanism for the current Track 2 story. Without transfer, the old dual-bank scaffold can remain frozen even when the degraded branch is nominally present. With transfer, the degraded bank finally receives sustained post-fault adaptation and becomes meaningfully different from the nominal path. This is why the paper should present the change as a bank-evolution mechanism rather than as a parameter tweak.

The second discussion point is maturity. The method is improved on average, but not uniformly across all case-metric pairs. The mild cases still show mixed behavior, and MPC remains stronger. These facts prevent the paper from claiming a final fault-tolerant controller. They do not, however, block a defensible paper. What they support is a more modest but still legitimate statement: the missing ingredient for a usable fault-aware dual-bank DeePC prototype was minimal warm-start / transfer.

The limitations are direct. Only a single fault family is tested. Only one onset time is used. Only two reference trajectories are used. No integrated fault detection or advanced switching logic is included. These constraints should remain visible in the final paper because they are part of the reason the current claim remains prototype-level.

Future work should remain narrow and subordinate to the current paper boundary. The obvious next technical target is to stabilize the mild-case regressions without reopening broad sweeps or changing the problem definition. After the current writeup is complete, a small targeted robustness check may be justified, but broad onset sweeps, parameter sweeps, new Track 2 variants, or fault-family expansion would move the project back into exploration mode and should stay out of the present paper.

## 7. Conclusion

This paper supports a conservative conclusion. Minimal warm-start / transfer is the mechanism that turns the current nominal/degraded DeePC scaffold from a frozen dual-bank construction into a fault-aware controller prototype under single-rotor degradation. The strengthened 10-seed matrix shows stable average gains over the old dual-bank baselines, while also exposing clear boundary cases in the mild settings. The method therefore deserves to be written as a prototype-level but defensible fault-aware DeePC improvement, not as a finalized controller and not as a method that surpasses MPC.
