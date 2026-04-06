# Introduction Skeleton

## Problem

DeePC is attractive for quadrotor outer-loop control because it can be data-driven and model-light, but fault-aware operation under actuator degradation is harder than nominal tracking.

## What breaks in the old scaffold

The nominal/degraded dual-bank scaffold looks reasonable at first, but without transfer the degraded bank can remain effectively coupled to the same data state as the nominal bank. In that case, the controller has a bank label, but not a genuinely different adaptive state.

## Why this matters

If the degraded bank does not keep adapting after fault onset, the scaffold does not yet support a strong fault-aware method claim.

## What we do

We add a minimal warm-start / transfer mechanism so the degraded bank can continue adapting after fault onset, while keeping the rest of the dual-bank structure unchanged.

## What we show

On a frozen single-rotor degradation benchmark with `step` and `figure8` trajectories, `mild` and `severe` fault settings, and fixed seeds, the transferred degraded bank improves average performance over the old nominal/degraded banks.

## Boundary

The method remains a lightweight prototype. It is not claimed to beat MPC, and it is not claimed to be a final controller.
