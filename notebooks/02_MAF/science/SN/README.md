# Rubin/LSST MAF — SN Ia science metrics

This folder contains a series of notebooks exploring the DESC/Rubin-LSST **Metrics Analysis Framework
(MAF)** metrics used to evaluate how well a given LSST cadence (an OpSim simulation) supports
**Type Ia supernova (SN Ia) science**. All metrics here answer variations of the same underlying
question: *"given the actual sequence of simulated LSST visits at a point on the sky, how well could
we discover, follow up, and cosmologically exploit Type Ia supernovae there?"*

They are numbered in a logical progression: `00`/`01`/`02` explore the flagship `SNNSNMetric`
(the DESC "number of SNe Ia" cadence metric) at increasing depth, while `03`/`04`/`05` explore three
lighter, complementary MAF metrics (`SNCadenceMetric`, `SNSNRMetric`, `SNSLMetric`) that isolate one
aspect each of what makes a cadence good for SN Ia.

## Scientific background

A cadence-quality metric for SNe Ia typically has to combine several ingredients:
- **Sampling** — how densely and regularly is a given sky position visited over a season?
- **Depth / SNR** — how bright must a SN be (at a given phase and redshift) to be detected at each
  visit, given the 5-sigma depth (`fiveSigmaDepth`), seeing, and sky brightness?
- **Light-curve completeness** — are there enough points before and after peak brightness, and in
  enough bands, to constrain a SALT2-like fit (time of maximum `t0`, stretch `x1`, color `c`)?
- **Cosmological reach** — combining the above with the SN Ia volumetric rate, how many *well-measured*
  SNe Ia does the survey yield out to some redshift limit `zlim`, and how far can that limit go?

Each notebook in this folder targets a different one (or combination) of these ingredients.

## Notebooks

### `00_SNIa_Fast.ipynb` — Quick sanity-check run of `SNNSNMetric`
Runs the flagship DESC metric `SNNSNMetric` with **all default parameters** on a very coarse HEALPix
grid (`nside=4`, 192 pixels over the whole sky) against the official LSST baseline cadence
(`get_baseline()`). Purely a fast end-to-end smoke test — not meant to produce science-quality numbers.
Explains, in a markdown cell, the 5 internal steps of the metric: simulate fake SNe Ia over a
redshift/peak-date grid → check which real visits would have observed each one before/after peak →
fit a simplified SALT2-like light curve → find `zlim` (redshift completeness limit, from a
color-uncertainty threshold) → combine `zlim` with the SN Ia rate to get `nSN` (expected number of
well-measured SNe Ia). Produces the two reduced per-pixel quantities `n_sn` and `zlim` and their
sky maps.

### `01_testSNIa.ipynb` — Detailed exploration of `SNNSNMetric`
Goes "under the hood" of `SNNSNMetric`:
- Sets the metric up with **explicit, non-default parameters** (`n_bef`, `n_aft`, `coadd_night`,
  `add_dust`, `hard_dust_cut`, `zmin`/`zmax`/`z_step`, `daymax_step`, `zlim_coeff`, `gamma_name`) and
  documents what each one controls, at a finer HEALPix resolution (`nside=16`).
- Bypasses the usual `MetricBundleGroup.run_all()` loop to call `metric.run(...)` **directly** on the
  visits of a single, densely-observed HEALPix pixel — much faster than a full-sky run, useful for
  debugging/timing.
- Builds several representative visit-sequence variants (dense/sparse pointing, one season with/without
  DDF visits, shallower depth, single-exposure visits) and saves them to `test_simData.hdf` for reuse
  as fixed regression-test inputs.
- Finally runs the metric over the **whole sky twice**: once on the WFD baseline survey, once on a
  Deep Drilling Field (DDF)-focused OpSim run (wider redshift range, since DDF visits are deeper), so
  the WFD vs DDF SN Ia yield/depth can be compared.

### `02_Number_SNeIa_metric.ipynb` — `nSN`/`zlim` summary values and the SN Ia rate model
Focuses on the final science numbers rather than the per-pixel mechanics:
- Runs `SNNSNMetric` with default parameters at two HEALPix resolutions (`nside=16` then `nside=8`) to
  check how sensitive the sky-averaged `n_sn`/`zlim` summary values are to pixel size.
- Repeats the `nside=8` run with Milky Way dust extinction enabled (`add_dust` default `True`) instead
  of disabled, to see its effect on the yield.
- Demonstrates `SnRate`, the standalone DESC SN Ia **volumetric rate model** used internally by
  `SNNSNMetric` to convert `zlim` into `nSN`, called directly (outside of MAF) and scaled to different
  survey areas (e.g. a single DDF pointing vs. the default area).

### `03_testSNCadenceMetric.ipynb` — `SNCadenceMetric`: pure sampling quality
Tests `SNCadenceMetric`, which measures the **quality of the temporal sampling alone** (visit spacing,
regularity, gaps) — independent of brightness, redshift, or noise. The metric converts the mean depth
(`m5`) and mean cadence (days between visits) of a slice into a reference redshift limit `zref`, via an
interpolation surface `lim_sn` (here a lightweight **synthetic/debug placeholder**, not a real SN
reference — replace it with a genuine simulated-SN-based `lim_sn` for scientific results). Includes a
`PatchedSNCadenceMetric` subclass that adapts the metric to modern `band`-based OpSim databases (v5.3+
store filter values like `r_57`, `g_6`, ... in the `filter` column, so `band` is used instead), and
compares three slicer choices: `UserPointsSlicer` on a single COSMOS-field point, `UserPointsSlicer` on
a handful of points, and a full-sky `HealpixSlicer`.

### `04_testSNSNRMMetric.ipynb` — `SNSNRMetric`: instrumental detectability
Tests `SNSNRMetric`, which estimates a **detection fraction** by comparing the signal-to-noise ratio
(SNR) of a simulated SN light curve (at a given redshift, using a luminosity/flux model) to the SNR of
synthetic ("fake") observations generated from each observing season's properties (`fiveSigmaDepth`,
cadence, season length). This introduces the instrumental physics (depth, seeing, sky brightness,
exposure time) that `SNCadenceMetric` deliberately ignores, but does not yet assess full light-curve
fit quality. Like notebook `03`, it patches the metric for `band`-based databases
(`PatchedSNSNRMetric`), uses a synthetic debug SN flux reference (`DemoSNSNRReference`), and compares
the same three slicer choices (single point, multiple points, full-sky HEALPix).

### `05_testSNSLMetric.ipynb` — `SNSLMetric`: light-curve usability
Tests `SNSLMetric`, which assesses whether a SN's light curve is **scientifically usable** for a
SALT2-like fit: enough useful points, sufficient coverage before/after maximum light, and multi-band
sampling to constrain `t0`, stretch, and color. This is the most complete of the three "lighter"
metrics, folding in cadence, SNR, and spectral (band) sampling together. Run here with a
`HealpixSlicer` at `nside=16` against the baseline cadence.

## Common conventions across these notebooks

- **Data location**: `RUBIN_SIM_DATA_DIR` environment variable points to the local cache of
  `rubin_sim`/`rubin_scheduler` auxiliary data (OpSim `.db` files, dust maps, SN gamma/noise files,
  throughputs). Falls back to `get_baseline()` for the official LSST baseline cadence, or to the first
  `.db` file found in the data directory.
- **Output**: each notebook creates its own temporary output directory (via `tempfile.TemporaryDirectory`)
  for MAF's `ResultsDb` SQLite bookkeeping database and any plotted figures — no shared state between
  notebooks.
- **`band` vs `filter`**: recent (v5.3+) OpSim databases store composite filter labels (e.g. `r_57`,
  `g_6`) in the `filter` column; the plain single-letter band (`u`, `g`, `r`, `i`, `z`, `y`) is in the
  `band` column instead. Notebooks `03`–`05` patch the installed metrics accordingly.
- **Plot options (`plot_dict`)**: MAF's plotting keyword dictionary uses **snake_case** keys in the
  current `rubin_sim` API (e.g. `percentile_clip`, `n_ticks`), not the legacy camelCase
  (`percentileClip`, `nTicks`) inherited from older `sims_maf`/`rubin_sim` notebook examples online.
- **Reduced quantities**: `SNNSNMetric` returns a compound per-pixel result that MAF's "reduce"
  mechanism splits into separate named bundles, `SNNSNMetric_reducen_sn` (expected SN Ia count) and
  `SNNSNMetric_reducezlim` (redshift completeness limit).
- **Debug/placeholder references**: in `03` and `04`, the `lim_sn` calibration surface is a synthetic
  stand-in built only to make the notebooks runnable end-to-end; it has no astrophysical meaning as-is
  and should be replaced with a real simulated-SN reference table for scientific results.

## Auxiliary file

- `test_simData.hdf`: HDF5 file produced by `01_testSNIa.ipynb`, containing several fixed visit-sequence
  scenarios (dense pointing, sparse pointing, one season with/without DDF visits, shallow depth,
  single-exposure visits) extracted from the baseline OpSim run, for reuse as regression-test inputs
  without re-querying the OpSim database.
