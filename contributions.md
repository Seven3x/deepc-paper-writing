# Contributions

1. A minimal warm-start / transfer bank-evolution mechanism that prevents post-fault freezing of the degraded DeePC bank relative to the old nominal/degraded scaffold.
2. A fixed single-rotor degradation matrix (2 trajectories x 2 severities x onset `20.0s` x 10 seeds) showing stable average improvement of `deepc_degraded_transfer` over the old dual-bank baselines.
3. A conservative boundary analysis that explicitly reports remaining regressions in mild cases and keeps MPC as a stronger reference baseline rather than a claim target.
