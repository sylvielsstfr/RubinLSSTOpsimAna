# 08_imagequality

Image quality (seeing) characterization for Rubin/LSST feature-scheduler cadence simulations,
using `rubin_sim.maf`. Complements the footprint/depth work in `../03_fbs5.3.6/` and
`../07_variateEVmV/` by looking at delivered seeing instead of visit counts or coadded depth.

Follows the repository convention (`data_<NN_TAG>/` for MAF outputs, `figs_<NN_TAG>/` for
figures, saved as PNG + PDF), as established in `../06_MAF_DESC_TaskF/`.

## Notebooks

- `01_Seeing_HealpixMaps.ipynb`
  Healpix maps (`nside=64`) and all-sky histograms of the median per-visit seeing
  (`seeingFwhmEff` and `seeingFwhmGeom`, `rubin_sim.maf.MedianMetric` + `HealpixSlicer`, plotted
  with `maf.HealpixSkyMap()` / `maf.HealpixHistogram()`), for the `baseline_v5.3.6_10yrs`
  simulation and one earlier baseline (`baseline_v5.3.5_10yrs` if present on disk, otherwise the
  next-older `baseline_v*.db` auto-discovered under `RUBIN_SIM_DATA_DIR`). Includes a
  side-by-side mosaic on a common color scale, a difference map between the two baselines,
  overlaid all-sky histograms, a per-band (`ugrizy`) breakdown, and a summary statistics table
  (mean/median/std/min/max), saved to CSV.
  Also repeats the map/histogram/mosaic/diff/overlay/summary treatment for the raw `airmass`
  column (same `HealpixSlicer(nside=64)`), then correlates the per-pixel `airmass` against
  `seeingFwhmEff`/`seeingFwhmGeom` for each run (scatter plot + per-run Pearson `r`,
  mean/median airmass), to check how much of the seeing map -- and of the seeing differences
  between runs -- traces back to the airmass distribution of the scheduled visits rather than a
  change in the underlying airmass-seeing relation.
  Outputs: `data_01_Seeing/`, `figs_01_Seeing/`.

## Column definitions

- `seeingFwhmEff`: effective seeing (arcsec), airmass-corrected, drives point-source photometric
  SNR / depth.
- `seeingFwhmGeom`: geometric/physical seeing (arcsec), airmass-corrected, relevant to
  astrometric precision and shape/PSF measurements (weak/strong lensing image quality).
  Both are per-visit columns already present in the opsim database (no stacker required); a
  Healpix median map mixes the atmospheric seeing model with the airmass distribution of visits
  across the sky.
- `airmass`: raw per-visit airmass (dimensionless, `sec(z)`), also already present in the opsim
  database. Analyzed directly (Section 9) and correlated per Healpix pixel against
  `seeingFwhmEff`/`seeingFwhmGeom` (Section 10) to disentangle how much of the seeing map -- and
  of the seeing differences between runs -- is inherited from where/how the scheduler points,
  rather than from a change in the underlying seeing model.

## Data

Simulations are auto-discovered from `baseline_v*_10yrs.db` files found under
`$RUBIN_SIM_DATA_DIR/sim_baseline/` (and `$RUBIN_SIM_DATA_DIR/` as a fallback). Edit the
`wanted` list in the first cells of `01_Seeing_HealpixMaps.ipynb` to compare a different pair
(or more) of runs, including non-baseline cadence families such as the
`shrink_fp_dust_*_v5.3.6_10yrs` runs used in `../07_variateEVmV/`.

## References

- `../03_fbs5.3.6/Footprint.ipynb` - `CountMetric`/`HealpixSlicer` pattern this series builds on.
- `../06_MAF_DESC_TaskF/`, `../07_variateEVmV/` - repository conventions (headers,
  `data_<TAG>/figs_<TAG>/`, dual PNG+PDF figure saving).
- `rubin_sim.maf` source: https://github.com/lsst/rubin_sim
- Table of simulations: https://usdf-maf.slac.stanford.edu/
- Opsim baseline runs: https://s3df.slac.stanford.edu/data/rubin/sim-data/sims_featureScheduler_runs5.3/baseline/
