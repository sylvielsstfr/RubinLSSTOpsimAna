# 10_DESCMAFDEPTHANDGALCOUNT

Detailed study of three `rubin_sim.maf` metrics used by the DESC static-probes and weak-lensing cases of the
SCOC (Survey Cadence Optimization Committee), **recomputed on cadence simulations whose footprint was
shrunk with a dust (E(B-V)) threshold**:

- `ExgalM5WithCuts`: usable extragalactic i-band depth (parent metric of the 3x2pt FoM),
- `GalaxyCountsMetricExtended` and `DepthLimitedNumGalMetric`: galaxy counts per pixel,
- `WeakLensingNvisits` and `RIZDetectionCoaddExposureTime`: weak-lensing systematics-mitigation proxies.

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
  `DepthLimitedNumGalMetric`. `nside = 64` (01, 03) and `128` (02).
- **Caching**: MAF is run once per (metric, simulation, E(B-V) cut) and the maps are saved as `.npz` in the
  `data_*` directory; the cache tag contains the cut actually used, so a change of `EBV_CUT_MODE` never
  reloads a map computed with another cut. `FORCE_RECOMPUTE = True` redoes everything.
- **Kernel**: `conda_py313_opsim53`.

## Points to check

- Notebook 03: the SQL band selection of `RIZDetectionCoaddExposureTime` is copied from the 06 demo notebook;
  check it against the docstring printed in Section 3 of the notebook.
- The dust map used by MAF (`DustMap`) is not necessarily the one used by the scheduler to build the
  footprint, so pixels along the footprint boundary may differ.

## References

- Lochner, M. et al. 2018, "Optimizing LSST Observing Strategy for Dark Energy Science", arXiv:1808.00006
- Awan, H. et al. 2016, ApJ 829, 50, "Testing LSST Dither Strategies for Survey Uniformity and Large-Scale Structure Systematics" (galaxy-count model)
- Zuntz, J. et al. 2021, "The LSST-DESC 3x2pt Tomography Optimization Challenge", arXiv:2108.13418
- Bianco, F. B. et al. 2022, ApJS 258, 1 (SCOC cadence optimization process)
- `rubin_sim.maf` source: https://github.com/lsst/rubin_sim
- `shrink_fp_dust_*.db` simulations: https://s3df.slac.stanford.edu/data/rubin/sim-data/sims_featureScheduler_runs5.3/shrink_fp/
