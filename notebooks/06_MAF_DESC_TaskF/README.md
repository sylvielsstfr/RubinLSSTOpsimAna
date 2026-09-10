# 06_MAF_DESC_TaskF

MAF evaluation of Rubin/LSST cadence simulations, restricted to the **DESC Task Force metrics** used by
the SCOC (Survey Cadence Optimization Committee): **3x2pt**, **Weak Lensing (WL)**, and **Supernovae (SN)**.

Each science topic gets its own numbered notebook, following the repository convention
(`data_<NN_TAG>/` for MAF outputs, `figs_<NN_TAG>/` for figures, saved as PNG + PDF).

## Notebooks

- `01_3x2pts_DESC_TaskForce_demo.ipynb`
  DESC 3x2pt static-probes Figure of Merit (galaxy clustering + weak lensing). Explains and runs the
  `rubin_sim.maf` implementation (`ExgalM5WithCuts` parent metric -> `StaticProbesFoMEmulatorMetricSimple`
  summary metric), matching the official `rubin_sim.maf.batches.science_radar_batch` "Cosmology" group.
  Produces Healpix maps + histograms of the usable extragalactic depth, and the effective area / depth /
  3x2pt FoM as a function of survey year, for `baseline_v5.3.6_10yrs.db`.
  Outputs: `data_01_3X2PTS/`, `figs_01_3X2PTS/`.

- WL (weak lensing) notebook: planned.
- SN (supernovae) notebook: planned.

## Data

Analyzed simulation: `/Users/dagoret/DATA/OpSim/sim_baseline/baseline_v5.3.6_10yrs.db`

## References

- Lochner, M. et al. 2018, "Optimizing LSST Observing Strategy for Dark Energy Science", arXiv:1808.00006
- Zuntz, J. et al. 2021, "The LSST-DESC 3x2pt Tomography Optimization Challenge", arXiv:2108.13418
- Bianco, F. B. et al. 2022, ApJS 258, 1 (SCOC cadence optimization process)
- `rubin_sim.maf` source: https://github.com/lsst/rubin_sim
