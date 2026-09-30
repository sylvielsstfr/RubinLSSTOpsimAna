# 10_DESCMAFDEPTHANDGALCOUNT

Detailed study of several `rubin_sim.maf` metrics used by the DESC static-probes, weak-lensing and
supernovae cases of the SCOC (Survey Cadence Optimization Committee), **recomputed on cadence simulations
whose footprint was shrunk with a dust (E(B-V)) threshold**:

- `ExgalM5WithCuts`: usable extragalactic i-band depth (parent metric of the 3x2pt FoM),
- effective surface area: the official `CountRatioMetric` summary statistic ("Effective Area") attached to `ExgalM5WithCuts` in `science_radar_batch` (00), and a seeing-weighted version built from the `n_eff` model (00b),
- `GalaxyCountsMetricExtended` and `DepthLimitedNumGalMetric`: galaxy counts per pixel,
- `WeakLensingNvisits` and `RIZDetectionCoaddExposureTime`: weak-lensing systematics-mitigation proxies,
- `StaticProbesFoMEmulatorMetric`: DESC 3x2pt static-probes Figure of Merit (GP emulator), year by year,
- `SNNSNMetric`: expected number of well-measured Type Ia supernovae (`n_sn`) and the redshift completeness limit (`zlim`),
- `KNePopMetric`: kilonova detection efficiency,
- `NestedLinearMultibandModelMetric` + `TomographicClusteringSigma8biasMetric`: DESC tomographic sigma8 bias (clustering systematics from non-uniform depth), year by year,
- `TdcMetric`: strong-lensing time-delay accuracy/precision/success rate (Time Delay Challenge) and the derived number of lenses and distance precision.

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

- `00_EffectiveSurfaceArea.ipynb`
  Effective surface area versus the E(B-V) cut with the **official** `rubin_sim` MAF, **no new metric**: the
  block *Cosmology / Static Science* of `rubin_sim/maf/batches/science_radar_batch.py` attaches
  `CountRatioMetric(norm_val=1/pix_area, metric_name='Effective Area (deg)')` to `ExgalM5WithCuts`, so
  `A_eff = N_valid_pixels x pixel_area` (the batch label says `deg`, the unit is deg^2). Batch settings for year 10:
  `i` band, `n_filters = 6`, `depth_cut = 25.9`, non-DDF visits, first 10 years, `nside = 64`,
  `DustMap(interp=False)`. `extinction_cut` is the threshold of each run (`USE_RUN_CUT = True`; `False` keeps the
  batch value 0.2 for all runs). Prints the source of `ExgalM5WithCuts` and `CountRatioMetric`, checks the summary
  value against `pixels x area`, tabulates `A_eff`, `A_eff / baseline` and the increment between consecutive
  thresholds, plots `A_eff` and `A_eff / A_eff(baseline)` versus the threshold, and draws the footprint maps (common
  color scale). Year-10 result: `A_eff` = 9499, 13100, 15515, 16523, 17455, 17525 and 17998 deg^2 for
  E(B-V) = 0.050, 0.080, 0.120, 0.150, 0.199, 0.200 (baseline) and 0.250, i.e. 0.542, 0.747, 0.885, 0.943, 0.996,
  1 and 1.027 of the baseline. The area is purely geometric (no seeing or source-density weighting) and requires
  coverage in ugrizy. Only year 10 is computed.
  Outputs: `data_00_EFFAREA/`, `figs_00_EFFAREA/`.

- `00b_EffectiveSurfaceArea_custom_neff.ipynb`
  Earlier, custom version of notebook 00: `EffectiveSurfaceAreaMetric`, a real MAF metric that packages the `n_eff`
  computation of notebook 05. In every HEALPix pixel it returns `a_eff = Omega_pix * n_eff / n_ref` (deg^2) with
  `n_ref = 27 arcmin^-2` (DESC SRD, year 10; it only rescales `A_eff`), so that the sum of the map is
  `A_eff = N_eff,tot / n_ref`. The footprint is defined inside the MAF (E(B-V) cut of the run and `i`-band coadded
  depth >= 25.9); the `n_eff` model is that of notebooks 04/05 (`generic` and `R2cut` variants). Section 7 checks
  the result against the caches of notebook 05.
  Outputs: `data_00b_EFFSURFACE/`, `figs_00b_EFFSURFACE/`.

- `DOC00_DepthsAndCountsMetrics.ipynb`
  Signature (`%pinfo`) and source (`%psource`) of `ExgalM5`, `GalaxyCountsMetricExtended` and
  `DepthLimitedNumGalMetric`, kept as a reference for the other notebooks (former `00_DepthsAndCountsMetrics.ipynb`).

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

- `10_Sigma8tomo_demo.ipynb`
  Tomographic sigma8-bias metric (Demo 1 of `../02_MAF/science/DESC/01_sigma8tomography_demo.ipynb`; the AreaAtRisk
  and mean-z demos of that notebook are not treated), recomputed for every year (1-10) on each of the 7 runs
  (`nside = 64`, `lmin = 10`, `power_multiplier = 0.1`, `n_filters = 6`). Two E(B-V)-dependent choices, both
  editable: `EBV_CUT_MODE` (`extinction_cut` of the parent metric, adapted to each run as in notebooks 01/03/06)
  and `FOOTPRINT_MODE` (the `lowdust` Healpix footprint of `SkyAreaGenerator` regenerated with `dust_limit` set to
  the threshold of each run, whereas the reference notebook uses a fixed footprint; the notebook checks that the
  installed `SkyAreaGenerator` accepts `dust_limit` and prints the footprint area per threshold). Bias versus year and
  versus E(B-V) threshold (raw and normalized to the baseline), heatmap over the (threshold, year) grid, footprint
  area versus threshold, and a consistency check against the reference notebook. Same layout as notebook 06.
  Outputs: `data_10_SIGMA8TOMO/`, `figs_10_SIGMA8TOMO/`.

- `11_TDC_TimeDelayAccuracy_demo.ipynb`
  `TdcMetric` (Liao et al. 2015 accuracy/precision/success-rate heuristics for time delays of lensed quasars) on the
  7 runs, from `../02_MAF/science/DESC/02_TDC_TimeDelayAccuracy.ipynb`. `TdcMetric` has no dust-cut parameter, so MAF
  is run once per simulation on the full sky (`nside = 64`, first 10 years, non-DDF) and the six quantities
  (accuracy, precision, rate, cadence, season, campaign) are cached as Healpix maps; the E(B-V) footprint is then a
  post-processing mask (`EBV_MASK_MODE`: `run`, `fixed` or `none`). Maps of accuracy and rate, high-accuracy region
  (A < 0.04 %), area-weighted histograms, maps of the differences between consecutive thresholds (with gained/lost
  pixels tabulated), and high-accuracy area, number of lenses, distance precision, mean rate and accuracy versus the
  E(B-V) threshold (raw and normalized). Both footprint-masked and all-valid-pixel statistics are tabulated.
  Outputs: `data_11_TDC/`, `figs_11_TDC/`.

- `100_MAFDESCComparison_demo.ipynb`
  Does **not** run MAF: reads the per-run summary tables cached by notebooks 00 to 11, reduces every MAF to one number
  per run, divides it by the value of the baseline run (E(B-V) = 0.2) and draws all the curves
  `MAF / MAF(baseline)` versus the E(B-V) threshold on the same axes. The legend is grouped by dependency on
  `ExgalM5WithCuts` (*direct*, *indirect*, *re-implemented cuts*, *none*; declared in `META` with a one-line
  justification and re-checked against the `rubin_sim` source when it can be imported). Curves: `ExgalM5WithCuts`
  (usable area of 01, effective area of 00, mean depth), `DepthLimitedNumGalMetric` and
  `GalaxyCountsMetricExtended` (total galaxies), `WeakLensingNvisits` (`gri`, `riz`) and
  `RIZDetectionCoaddExposureTime`, `N_eff` of 05 (`generic`, `R2cut`), 3x2pt FoM at year 10, `SNNSNMetric`
  (`n_sn`, `zlim`), `KNePopMetric` detections, sigma8 bias at year 10, `TdcMetric` (number of lenses, distance
  precision). Section 4.1 splits the plot by dependency category; Section 4.2 shows the total effective number of
  galaxies `N_eff,tot` (integral of `n_eff` over the footprint, and its factorization into footprint area times mean
  `n_eff`); Section 4.3 shows the effective area of notebook 00 (cross-check against the usable area of notebook 01,
  comparison with the weak-lensing footprint area and `N_eff,tot`); Section 5 is a summary table.
  A curve whose cached table is missing is skipped and reported: run the corresponding notebook first.
  Reads: `data_00_EFFAREA/`, `data_01_EXGALM5CUTS/`, `data_02_GALCOUNTS/`, `data_03_WL/`, `data_05_NEFFMAPS/`,
  `data_06_FOM3X2PTS/`, `data_07_SNCOUNTS/`, `data_09_KNE/`, `data_10_SIGMA8TOMO/`, `data_11_TDC/`.
  Outputs: `data_100_MAFCOMPARISON/`, `figs_100_MAFCOMPARISON/`.

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

  Notebooks 00 and 00b select the baseline as `baseline_v5.3.6_10yrs` (looked up in `OPSIM_DIR` and in
  `OPSIM_DIR/sim_baseline/`), and notebook 100 reads cached files tagged `baseline_v5_3_6_10yrs`, whereas the table
  lists `baseline_v5.3.6_11yrs.db` (truncated to 10 years by the `night` cut). Check which baseline file is used
  and update the table if the 10-year file is now the one used everywhere.

- **E(B-V) cut of the metrics adapted to each run.** In the three notebooks, `EBV_CUT_MODE = 'run'` (default)
  gives the metric the E(B-V) threshold that defines the WFD (Wide Fast Deep) footprint of the run
  (`extinction_cut` for `ExgalM5WithCuts`, `lim_ebv` for `DepthLimitedNumGalMetric`, `ebvlim` for the WL
  metrics). `EBV_CUT_MODE = 'fixed'` uses the same cut `FIXED_LIM_EBV = 0.2` for all runs. The differences
  between consecutive maps combine the change of the metric footprint and the change of the visit
  distribution produced by the scheduler. `GalaxyCountsMetricExtended` has no E(B-V) cut. Notebook 00 follows the same convention with
  `USE_RUN_CUT = True` (`extinction_cut` = threshold of each run; `False` keeps the batch value 0.2).
- **Visits**: first 10 years of every run (`night <= 10*365.25 + 0.5`, needed because the baseline file has 11
  years), non-DDF (`scheduler_note not like 'DD%'`).
- **Depth cut**: `25.9` in notebooks 00, 00b, 01 and 03 (year-10 value of the official batch), `26.0` inside
  `DepthLimitedNumGalMetric`, per-year values (`MAG_CUTS`) in notebook 06. `nside = 64` (00, 00b, 01, 03, 06), `128`
  (02), `32` (07 - `SNNSNMetric` is too expensive per pixel for `nside = 64`).
- **Caching**: MAF is run once per (metric, simulation, E(B-V) cut) and the maps are saved as `.npz` in the
  `data_*` directory; the cache tag contains the cut actually used, so a change of `EBV_CUT_MODE` never
  reloads a map computed with another cut. `FORCE_RECOMPUTE = True` redoes everything.
- **Kernel**: `conda_py313_opsim53`.
- **Notebooks 10 and 11**: not part of the original three-notebook E(B-V) convention; `TdcMetric` (11) has no E(B-V)
  cut of its own and notebook 10 also acts on the slicer footprint (see their descriptions above).

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

- Notebook 10: the footprint follows the run only if `SkyAreaGenerator` accepts `dust_limit` and if the
  `lowdust` region of the regenerated footprint reproduces the WFD of the `shrink_fp_dust` runs; check the footprint
  table in Section 4 of the notebook. The dlogN/dm5 model and the Cell fits of `DENSITY_TOMOGRAPHY_MODEL` are not
  re-derived for the shrunk footprints.
- Notebook 11: the E(B-V) mask of the footprint is a post-processing approximation of the scheduler footprint; the
  reference notebook uses all visits and no mask, this one the first 10 years, non-DDF visits.
- Notebook 00: with `USE_RUN_CUT = True` the curve mixes the change of the simulated footprint and the change of the
  metric cut (`USE_RUN_CUT = False` isolates the footprint). The area requires coverage in ugrizy (`n_filters = 6`),
  so it is smaller than the `gri` weak-lensing footprint of notebook 03. Its introduction and caveats refer to
  `00_EffectiveSurfaceArea_custom_neff_old.ipynb`, which is now `00b_EffectiveSurfaceArea_custom_neff.ipynb`; the
  overview of 00b still quotes `data_00_EFFSURFACE/`, `figs_00_EFFSURFACE/` and
  `00_DepthsAndCountsMetrics_old.ipynb`, whereas its code writes to `data_00b_EFFSURFACE/`, `figs_00b_EFFSURFACE/` and
  the old content is now `DOC00_DepthsAndCountsMetrics.ipynb`.
- Notebook 100: the dependency categories are declarations (`META`); Section 2.1 only re-checks them if `rubin_sim`
  is importable in the kernel. The effective area of 00b is not read by this notebook.

## References

- Lochner, M. et al. 2018, "Optimizing LSST Observing Strategy for Dark Energy Science", arXiv:1808.00006
- Awan, H. et al. 2016, ApJ 829, 50, "Testing LSST Dither Strategies for Survey Uniformity and Large-Scale Structure Systematics" (galaxy-count model)
- Zuntz, J. et al. 2021, "The LSST-DESC 3x2pt Tomography Optimization Challenge", arXiv:2108.13418
- Gris, Ph. et al. 2023, ApJS 264, 22, "Designing an Optimal LSST Deep Drilling Program for Cosmology with Type Ia Supernovae" (SN selection criteria used by `SNNSNMetric`)
- Bianco, F. B. et al. 2022, ApJS 258, 1 (SCOC cadence optimization process)
- `rubin_sim.maf` source: https://github.com/lsst/rubin_sim
- `shrink_fp_dust_*.db` simulations: https://s3df.slac.stanford.edu/data/rubin/sim-data/sims_featureScheduler_runs5.3/shrink_fp/
