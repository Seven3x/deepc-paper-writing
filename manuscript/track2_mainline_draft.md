# Track 2 Mainline Draft

This Markdown draft is superseded by:

- `paper/manuscript/track2_mainline_draft.tex`

Current paper-facing facts are:

- The main 10-seed matrix is DeePC-family only:
  - `deepc_nominal_bank`
  - `deepc_degraded_bank`
  - `deepc_degraded_transfer`
- External controller comparison is now handled separately through:
  - `identified_mpc`
- The paper no longer uses the old `mpc` results in the manuscript, tables, or figure captions.
- The current defensible result boundary is:
  - frozen dual-bank diagnosis
  - transfer necessity mechanism
  - clear local gain in `step + mild`
  - mixed behavior elsewhere
  - fair identified-model MPC stronger on `step`, but unstable on `figure8`

Use the TeX manuscript as the only authoritative draft.
