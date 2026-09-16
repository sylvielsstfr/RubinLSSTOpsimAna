# 07_variateEVmV

Impact of the dust-extinction cutoff threshold, **E(B-V)**, used to define the WFD footprint
boundary, on the Rubin/LSST v5.3.6 feature-scheduler cadence simulations. The `shrink_fp_dust`
family of runs shrinks or grows the WFD footprint by moving this dust cut, trading survey area
against depth/number of visits near the Galactic plane; this notebook series quantifies that
trade-off directly on the Healpix maps that feed the SCOC "SRD" Figure of Merit.

Follows the repository convention (`data_<NN_TAG>/` for MAF outputs, `figs_<NN_TAG>/` for
figures, saved as PNG + PDF), as established in `../06_MAF_DESC_TaskF/`.

## Notebooks

- `01_FOMNv_HealpixDiff_ShrinkFPDust.ipynb`
  `FOMNv` Healpix maps and pairwise differences. Computes (or reuses cached results from
  `../03_fbs5.3.6/`) the per-pixel `NVisits` Healpix map (`rubin_sim.maf.batches.fOBatch`,
  bundle `"..._fO_All_visits_HEAL"`, `nside=64`, all visits/bands/years) - the map that directly
  underlies the official scalar `fONv` Figure of Merit (number of visits reached at the 825 sq deg
  reference area). Plots all 7 dust-threshold variants side by side, then plots and saves the
  difference map for each pair of *consecutive* thresholds: `0.080-0.050`, `0.120-0.080`,
  `0.150-0.120`, `0.199-0.150`, `baseline-0.199`, `0.250-baseline`. Adds a summary table
  (mean/median/std/min/max per pair) and histograms, plus an optional cross-check of the scalar
  `fONv` values from `summary.h5`.
  Outputs: `data_01_FOMNV/`, `figs_01_FOMNV/`.

## Data

Simulations analyzed (`/Users/dagoret/DATA/OpSim/`), sorted by increasing dust threshold:

| Run | E(B-V) threshold |
|---|---|
| `shrink_fp_dust_0.050_v5.3.6_10yrs.db` | 0.050 |
| `shrink_fp_dust_0.080_v5.3.6_10yrs.db` | 0.080 |
| `shrink_fp_dust_0.120_v5.3.6_10yrs.db` | 0.120 |
| `shrink_fp_dust_0.150_v5.3.6_10yrs.db` | 0.150 |
| `shrink_fp_dust_0.199_v5.3.6_10yrs.db` | 0.199 |
| `baseline_v5.3.6_10yrs.db` (= `shrink_fp_dust_0.200_v5.3.6_10yrs.db`, under `sim_baseline/`) | 0.200 |
| `shrink_fp_dust_0.250_v5.3.6_10yrs.db` | 0.250 |

## References

- `../03_fbs5.3.6/Footprint.ipynb` - `CountMetric`/`HealpixSlicer` definition of `NVisits`, and
  the `fOBatch` FP-comparison section this series builds on.
- `../06_MAF_DESC_TaskF/` - repository conventions (headers, `data_<TAG>/figs_<TAG>/`, dual
  PNG+PDF figure saving).
- `rubin_sim.maf.batches.fOBatch` source: https://github.com/lsst/rubin_sim/blob/main/rubin_sim/maf/batches/srd_batch.py
- `shrink_fp_dust_*.db` simulations: https://s3df.slac.stanford.edu/data/rubin/sim-data/sims_featureScheduler_runs5.3/shrink_fp/
- Table of simulations: https://usdf-maf.slac.stanford.edu/
- `rubin_sim.maf` source: https://github.com/lsst/rubin_sim
