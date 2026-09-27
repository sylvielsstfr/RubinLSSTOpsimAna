# 10_DESCMAFDEPTHANDGALCOUNT

Detailed study of several `rubin_sim.maf` metrics used by the DESC static-probes, weak-lensing and
supernovae cases of the SCOC (Survey Cadence Optimization Committee), **recomputed on cadence simulations
whose footprint was shrunk with a dust (E(B-V)) threshold**:

- `ExgalM5WithCuts`: usable extragalactic i-band depth (parent metric of the 3x2pt FoM),
- `GalaxyCountsMetricExtended` and `DepthLimitedNumGalMetric`: galaxy counts per pixel,
- `WeakLensingNvisits` and `RIZDetectionCoaddExposureTime`: weak-lensing systematics-mitigation proxies,
- `StaticProbesFoMEmulatorMetric`: DESC 3x2pt static-probes Figure of Merit (GP emulator), year by year,
- `SNNSNMetric`: expected number of well-measured Type Ia supernovae (`n_sn`) and the redshift completeness limit (`zlim`).

The goal is to understand precisely how these metrics are computed and where they differ, in order to
improve them later (PSF handling, weak-lensing metric).

The presentation follows `../07_variateEVmV/01_FOMNv_HealpixDiff_ShrinkFPDust.ipynb`, and the workflow
conventions of `../06_MAF_DESC_TaskF` (`data_<NN_TAG>/` for MAF outputs and cached maps, `figs_<NN_TAG>/` for
figures, saved as PNG + PDF). For every metric the notebooks produce:

1. Healpix maps for the 7 simulations, on one common color scale,
2. area-weighted histograms of the valid pixels,
3. maps of the **differences between consecutive E(B-V) thresholds**
   (`0.080-0.050`, `0.120-0.080`, `0.150-0.120`, `0.199-0.150`, `baseline-0.199`, `0.250-baseline`),
   with the pixels gained or lost by the footprint shown separately and summary tables.

## Notebooks

- `00_DepthsAndCountsMetrics.ipynb`
  Signature (`%pinfo`) and source (`%psource`) of `ExgalM5`, `GalaxyCountsMetricExtended` and
  `DepthLimitedNumGalMetric`, kept as a reference for the other notebooks.

- `01_compareExgalM5withCuts.ipynb`
  `ExgalM5WithCuts` (coadded, dust-corrected i-band depth, masked where the dust, 6-band coverage or depth
  cuts fail) on the 7 runs. Also studies, without rerunning MAF, the effect of a stricter depth cut
  (25.9 to 26.2) on the usable area, and gives notes for a PSF-aware version of the metric.
  Outputs: `data_01_EXGALM5CUTS/`, `figs_01_EXGALM5CUTS/`.

- `02_compareGalaxyCounts.ipynb`
  `GalaxyCountsMetricExtended` versus `DepthLimitedNumGalMetric` (`nside = 128`, i band, all redshifts):
  how each count is computed, where they differ (footprint cuts, upper magnitude limit), with a
  **single-pixel experiment** run on the real metric objects (count versus coadded depth, sensitivity to the
  0.7 mag offset hard-wired in `DepthLimitedNumGalMetric`). On the 7 runs, the total of
  `GalaxyCountsMetricExtended` is split into a footprint effect and a truncation effect.
  Outputs: `data_02_GALCOUNTS/`, `figs_02_GALCOUNTS/`.

- `03_WeakLensing.ipynb`
  `WeakLensingNvisits` (`gri` and `riz`) and `RIZDetectionCoaddExposureTime` on the 7 runs, plus a `gri` versus
  `riz` comparison on the baseline and the totals and footprint area as a function of the threshold, and the mean and median number of visits and exposure time, with the pixel-to-pixel dispersion, as a function of the threshold. The
  metric calls are those of `../06_MAF_DESC_TaskF/02_WL_DESC_TaskForce_demo.ipynb`, which covers the baseline
  year by year.
  Outputs: `data_03_WL/`, `figs_03_WL/`.

- `04_NeffSeeingModel.ipynb`
  Standalone (no OpSim database, no `rubin_sim`) test of a seeing- and depth-dependent model of the weak-lensing
  effective source density `n_eff` (Chang et al. 2013), a first step towards a seeing-aware weak-lensing MAF
  metric. The model is evaluated with the seeing and depth of HSC-Y3, KiDS-1000 and DES-Y3 and compared with
  their published `n_eff`; its sensitivity to the assumed galaxy population is estimated; the dependence on
  seeing and depth is tabulated for LSST-like coadds (grid saved for later use in a MAF metric).
  Outputs: `data_04_NEFF/`, `figs_04_NEFF/`.

- `05_NeffMaps.ipynb`
  Applies the model of notebook 04, pixel by pixel, to the 7 simulations: coadded depth and effective seeing
  in `r` and `i` (computed with MAF: `ExgalM5WithCuts` with all cuts disabled, and a custom `sqrt(mean(FWHM^2))`
  metric), restricted to the WL footprint of notebook 03, feed a vectorized version of the `n_eff` model in two
  variants (`generic`, and `R2cut` with the HSC-like resolution cut). Includes a consistency check of the depth
  against notebook 01, maps and histograms of `n_eff`, differences between consecutive thresholds, `N_eff`
  (effective number of galaxies of the footprint) versus the dust threshold, and a check of what the visit-count
  proxy of notebook 03 misses relative to the seeing. First prototype of a seeing-aware weak-lensing MAF metric;
  Section 12 lists what is still needed (PSF systematics term, angular power spectrum, absolute calibration).
  Outputs: `data_05_NEFFMAPS/`, `figs_05_NEFFMAPS/`.

- `06_FOM3x2pts_DESC_TaskForce_demo.ipynb`
  `StaticProbesFoMEmulatorMetric` (GP emulator) and `StaticProbesFoMEmulatorMetricSimple` (cross-check),
  recomputed for every year (1-10) on each of the 7 runs (`ExgalM5WithCuts` chain, `nside = 64`, per-year
  `depth_cut` from the official batch, `extinction_cut` adapted to each run as in notebooks 01/03). FoM versus
  year and versus E(B-V) threshold (raw and normalized), a heatmap over the full (threshold, year) grid, and a
  consistency check against notebook 01 at year 10. Reference notebook:
  `../06_MAF_DESC_TaskF/01b_3x2pts_DESC_TaskForce_demo.ipynb`.
  Outputs: `data_06_FOM3X2PTS/`, `figs_06_FOM3X2PTS/`.

- `07_SNCounts_DESC_TaskForce_demo.ipynb`
  `SNNSNMetric` (`n_sn`, `zlim`) on the 7 runs, at `nside = 32` (coarser than the `nside = 64` used elsewhere in
  this series but finer than the official `nside = 16`, chosen because the metric is much slower per pixel; one
  MAF run per simulation, no year loop, as in the official batch). Dust cut: the metric's own `hard_dust_cut`,
  adapted to each run as `extinction_cut`/`lim_ebv`/`ebvlim` are in notebooks 01/03/06. Healpix maps,
  histograms, differences between consecutive thresholds (`n_sn` filled with 0 where masked, `zlim` restricted
  to pixels valid in both runs), and total `n_sn` / mean-median `zlim` versus the E(B-V) threshold. Reference
  notebook: `../06_MAF_DESC_TaskF/03_SN_DESC_TaskForce_demo.ipynb`.
  Outputs: `data_07_SNCOUNTS/`, `figs_07_SNCOUNTS/`.

- `08_SNCounts_ReplotFromCache.ipynb`
  Regenerates every figure and table of notebook 07 **by reading back its cached `.npz` maps**, without
  rerunning `SNNSNMetric` (much faster - `SNNSNMetric` is slow, so this notebook lets figures/captions be
  tweaked without redoing the MAF computation). Same sections and text as notebook 07; raises a clear
  `FileNotFoundError` if a run is missing from the cache (run notebook 07 for it first).
  Reads: `data_07_SNCOUNTS/`. Outputs: `data_08_SNCOUNTS_REPLOT/`, `figs_08_SNCOUNTS_REPLOT/`.

- `09_KNe_DESC_TaskForce_demo.ipynb`
  `KNePopMetric` (kilonova detection efficiency; single GW170817-like model and the full Bulla model grid) on
  the 7 runs. Unlike the other metrics of this series, `KNePopMetric` has no dust-cut parameter of its own and
  uses a `UserPointsSlicer` (scattered injected events, not Healpix): the **same injected population (fixed
  seed)** is reused for every run, so differences reflect only the cadence/footprint. Aitoff sky-map scatter
  plots per run, detection efficiency (all 7 criteria) versus the E(B-V) threshold and its normalized version,
  event-by-event "gained/lost detection" transition maps between consecutive thresholds, and a consistency
  check against the reference notebook. Reference notebook: `../06_MAF_DESC_TaskF/04_KNe_DESC_TaskForce_demo.ipynb`.
  Outputs: `data_09_KNE/`, `figs_09_KNE/`.

## Common choices

- **Simulations** (`/Users/dagoret/DATA/OpSim/`, all v5.3.6):

  | run | E(B-V) threshold of the footprint |
  |---|---|
  | `shrink_fp_dust_0.050_v5.3.6_10yrs.db` | 0.050 |
  | `shrink_fp_dust_0.080_v5.3.6_10yrs.db` | 0.080 |
  | `shrink_fp_dust_0.120_v5.3.6_10yrs.db` | 0.120 |
  | `shrink_fp_dust_0.150_v5.3.6_10yrs.db` | 0.150 |
  | `shrink_fp_dust_0.199_v5.3.6_10yrs.db` | 0.199 |
  | `baseline_v5.3.6_11yrs.db` | 0.200 |
  | `shrink_fp_dust_0.250_v5.3.6_10yrs.db` | 0.250 |

- **E(B-V) cut of the metrics adapted to each run.** In the three notebooks, `EBV_CUT_MODE = 'run'` (default)
  gives the metric the E(B-V) threshold that defines the WFD (Wide Fast Deep) footprint of the run
  (`extinction_cut` for `ExgalM5WithCuts`, `lim_ebv` for `DepthLimitedNumGalMetric`, `ebvlim` for the WL
  metrics). `EBV_CUT_MODE = 'fixed'` uses the same cut `FIXED_LIM_EBV = 0.2` for all runs. The differences
  between consecutive maps combine the change of the metric footprint and the change of the visit
  distribution produced by the scheduler. `GalaxyCountsMetricExtended` has no E(B-V) cut.
- **Visits**: first 10 years of every run (`night <= 10*365.25 + 0.5`, needed because the baseline file has 11
  years), non-DDF (`scheduler_note not like 'DD%'`).
- **Depth cut**: `25.9` in notebooks 01 and 03 (year-10 value of the official batch), `26.0` inside
  `DepthLimitedNumGalMetric`, per-year values (`MAG_CUTS`) in notebook 06. `nside = 64` (01, 03, 06), `128`
  (02), `32` (07 - `SNNSNMetric` is too expensive per pixel for `nside = 64`).
- **Caching**: MAF is run once per (metric, simulation, E(B-V) cut) and the maps are saved as `.npz` in the
  `data_*` directory; the cache tag contains the cut actually used, so a change of `EBV_CUT_MODE` never
  reloads a map computed with another cut. `FORCE_RECOMPUTE = True` redoes everything.
- **Kernel**: `conda_py313_opsim53`.

## Points to check

- Notebook 03: the SQL band selection of `RIZDetectionCoaddExposureTime` is copied from the 06 demo notebook;
  check it against the docstring printed in Section 3 of the notebook.
- The dust map used by MAF (`DustMap`) is not necessarily the one used by the scheduler to build the
  footprint, so pixels along the footprint boundary may differ.
- Notebook 06: `StaticProbesFoMEmulatorMetric` is a GP trained on a 36-point (area, depth) grid; early-year
  and/or small-footprint cells can fall near or outside that grid and are then extrapolations.
- Notebook 07: `hard_dust_cut`'s official default is `0.25`, not the `0.2` used as `FIXED_LIM_EBV` elsewhere in
  this series; the notebook uses `scheduler_note not like 'DD%'` while the reference `03_SN_DESC_TaskForce_demo.ipynb`
  uses the wider `'%DD%'` - check the DDF exclusion if the two notebooks disagree.

## References

- Lochner, M. et al. 2018, "Optimizing LSST Observing Strategy for Dark Energy Science", arXiv:1808.00006
- Awan, H. et al. 2016, ApJ 829, 50, "Testing LSST Dither Strategies for Survey Uniformity and Large-Scale Structure Systematics" (galaxy-count model)
- Zuntz, J. et al. 2021, "The LSST-DESC 3x2pt Tomography Optimization Challenge", arXiv:2108.13418
- Gris, Ph. et al. 2023, ApJS 264, 22, "Designing an Optimal LSST Deep Drilling Program for Cosmology with Type Ia Supernovae" (SN selection criteria used by `SNNSNMetric`)
- Bianco, F. B. et al. 2022, ApJS 258, 1 (SCOC cadence optimization process)
- `rubin_sim.maf` source: https://github.com/lsst/rubin_sim
- `shrink_fp_dust_*.db` simulations: https://s3df.slac.stanford.edu/data/rubin/sim-data/sims_featureScheduler_runs5.3/shrink_fp/
