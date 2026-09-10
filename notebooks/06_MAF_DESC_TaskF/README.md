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

- `01b_3x2pts_DESC_TaskForce_demo.ipynb`
  Variant of notebook 01 that replaces `StaticProbesFoMEmulatorMetricSimple` with
  `StaticProbesFoMEmulatorMetric` (Gaussian-Process emulator, via `george`), the summary metric actually
  used by the official `science_radar_batch` (rather than the bilinear-interpolation "Simple" version).
  Unlike the Simple emulator (years 1/3/6/10 only), the GP has no `year` argument and is evaluated at every
  year 1-10; both are run side by side for a direct comparison plot. Same `ExgalM5WithCuts` depth maps as
  notebook 01 (not re-plotted in full here).
  Outputs: `data_01b_3X2PTS/`, `figs_01b_3X2PTS/`.

- `02_WL_DESC_TaskForce_demo.ipynb`
  DESC Weak Lensing systematics-mitigation proxy metrics. Explains and runs `WeakLensingNvisits` (visits per
  pixel over the reduced, dust/depth-cut footprint, in `gri` and `riz`) and `RIZDetectionCoaddExposureTime`
  (total `riz` detection-coadd exposure time), matching the official `science_radar_batch` "WL" subgroup.
  Full-survey headline number plus per-year (1-9) Healpix maps, histograms and trend plots.
  Outputs: `data_02_WL/`, `figs_02_WL/`.

- `03_SN_DESC_TaskForce_demo.ipynb`
  DESC Supernovae Task Force metric: counts the expected number of well-measured Type Ia SNe using
  `SNNSNMetric` (fast SALT2-like light-curve simulation -> redshift completeness limit `zlim` -> SN count
  `n_sn` via a volumetric rate model), matching the official `science_radar_batch` "SNe Ia" subgroup
  (WFD only, `nside=16`, 0.2 <= z <= 0.5). Reports the headline total (`"Total detected"`) plus Healpix
  maps/histograms of `n_sn` and `zlim`.
  Outputs: `data_03_SN/`, `figs_03_SN/`.

## Data

Analyzed simulation: `/Users/dagoret/DATA/OpSim/sim_baseline/baseline_v5.3.6_10yrs.db`

## References

- Lochner, M. et al. 2018, "Optimizing LSST Observing Strategy for Dark Energy Science", arXiv:1808.00006
- Zuntz, J. et al. 2021, "The LSST-DESC 3x2pt Tomography Optimization Challenge", arXiv:2108.13418
- Gris, Ph. et al. 2023, "Designing an Optimal LSST Deep Drilling Program for Cosmology with Type Ia Supernovae", ApJS 264, 22
- Bianco, F. B. et al. 2022, ApJS 258, 1 (SCOC cadence optimization process)
- `rubin_sim.maf` source: https://github.com/lsst/rubin_sim
