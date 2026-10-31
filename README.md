# RubinLSSTOpsimAna

[![Template](https://img.shields.io/badge/Template-LINCC%20Frameworks%20Python%20Project%20Template-brightgreen)](https://lincc-ppt.readthedocs.io/en/latest/)
[![PyPI](https://img.shields.io/pypi/v/rubinlsstopsimana?color=blue&logo=pypi&logoColor=white)](https://pypi.org/project/rubinlsstopsimana/)
[![GitHub Workflow Status](https://img.shields.io/github/actions/workflow/status/LSSTDESC/rubinlsstopsimana/smoke-test.yml)](https://github.com/LSSTDESC/rubinlsstopsimana/actions/workflows/smoke-test.yml)
[![Codecov](https://codecov.io/gh/LSSTDESC/rubinlsstopsimana/branch/main/graph/badge.svg)](https://codecov.io/gh/LSSTDESC/rubinlsstopsimana)
[![Read The Docs](https://img.shields.io/readthedocs/rubinlsstopsimana)](https://rubinlsstopsimana.readthedocs.io/)
[![Benchmarks](https://img.shields.io/github/actions/workflow/status/LSSTDESC/rubinlsstopsimana/asv-main.yml?label=benchmarks)](https://LSSTDESC.github.io/rubinlsstopsimana/)

## Purpose

This package studies the metrics of [`rubin_sim`](https://github.com/lsst/rubin_sim) and its **Metrics
Analysis Framework (MAF)**, and exercises them against Rubin/LSST feature-scheduler cadence simulations
(OpSim outputs). It reproduces, adapts, and runs the metrics distributed with `rubin_sim` and with the
companion example notebooks in [`rubin_sim_notebooks`](https://github.com/lsst/rubin_sim_notebooks), on
the author's own machine and against the author's own simulation files.

The notebooks kept in this repository are restricted to what has been **personally verified to run**:
each numbered notebook has been executed end-to-end at least once, and any upstream bug found along the
way (e.g. a typo in a `rubin_sim.maf` cosmology summary metric) has been reported/fixed rather than
silently worked around.

Notebooks live under `notebooks/`, in a series of numbered subdirectories that roughly follow the order
in which the corresponding topics were explored:

| Directory | Topic |
|---|---|
| `01_ExploreSims` | First contact with OpSim `.db` files (no MAF yet) |
| `02_MAF` | Learning and validating MAF itself: tutorials + per-science-case metrics |
| `03_fbs5.3` | Full metric evaluation on the `baseline_v5.3.0_10yrs` simulation |
| `03_fbs5.3.6` | Same evaluation on `baseline_v5.3.6_10yrs`, incl. footprint/`fO` comparisons |
| `04_fbs5.3_SN` | `SNNSNMetric` (DESC SN Ia count) on `baseline_v5.3.5_10yrs` |
| `05_rubinsimutil_SN` | Exploring the low-level SN utility functions used by `SNNSNMetric` |
| `06_MAF_DESC_TaskF` | DESC/SCOC metrics restricted to 3x2pt, WL, SN, KNe, Astrometry |
| `07_variateEVmV` | Effect of the WFD dust-extinction cutoff on the footprint (SCOC Footprint Task Force) |
| `08_imagequality` | Image quality: seeing (and airmass) Healpix maps and histograms, baseline v5.3.6 vs v5.3.5 |
| `09_RubinMaps` | Inspection of the `rubin_sim` HEALPix maps: dust (2D/3D) and stellar density |
| `10_DESCMAFDEPTHANDGALCOUNT` | DESC MAF metrics (depth, galaxy counts, WL, 3x2pt, SN, KNe, sigma8, TDC) vs the E(B-V) cut of the footprint, 7 runs |
| `10_DESCMAFDEPTHANDGALCOUNT_FootpTF` | Same series as `10_...`, extended to 10 runs (extra E(B-V) = 0.090, 0.100, 0.110) |
| `11_NEWDESCMAF` | Draft of new DESC MAF metrics with a tunable E(B-V) cut (`DustLimitedNumGalMetric`, `EBVAreaMetric`) |
| `12_SurveyStrategy` | Survey modes, areas, DDF/WFD timelines, filter balance, multiplets, modified footprint (adapted from the official survey-strategy notebooks) |

## Notebook directories

### `01_ExploreSims`
Direct exploration of OpSim simulation databases with `rubin_sim`/pandas/sqlite, with no MAF involved
yet: reading a `.db` file and plotting the standard per-visit quantities (airmass, seeing, sky
brightness, moon/sun geometry, slew, ...), comparing two simulations run-by-run (e.g. `observation_reason`,
`scheduler_note`, `science_program`, `target_name` distributions), and producing summary statistics
per photometric band.

### `02_MAF`
Learning and validating MAF itself, organized in four parts:
- `tutorial/` — reproduction of the official MAF tutorial series (introduction to MAF, writing metrics,
  plotting, getting data/summary metrics for whole families of runs, a single-point walkthrough, survey
  footprint), checked to run locally.
- `opsimdb/`, `hourglass/` — dumping and summarizing OpSim visit metadata, and "hourglass" plots of
  visit activity vs. time of night and time of year.
- `science/` — MAF metrics organized by science case, each one run against a real baseline simulation:
  `AGN` (quasar number counts, structure function, time lags), `DESC` (3x2pt sigma8 tomography demo,
  time-delay-cosmography accuracy), `Galaxy` (bulge distances, galactic-plane metrics), `KiloNovae`,
  `LocalVolume` (dwarf galaxies), `MicroLensing`, `SN` (a full progression of Type Ia supernova cadence
  metrics — see its own `README.md`), `SSO` (solar system objects), `Stars` (periodic/pulsating stars,
  young stellar objects), `TDE`, `XRB`, and `Technical` (generic MAF mechanics: a new-metric workbook,
  single-point visualization, filter-pair metrics, seasons, time gaps, `teff`).

### `03_fbs5.3`
Full evaluation of the standard science-radar metric set on `baseline_v5.3.0_10yrs`: overall `NVisits`
Healpix maps, per-year `NVisits` maps (years 1-10, with/without ToO and DDF visits), and a "v5.3 Update"
summary-grid notebook (`v5.3_Update.ipynb`).

### `03_fbs5.3.6`
The same evaluation on `baseline_v5.3.6_10yrs` — the update that reset the survey-forecast weather years
to a subset weighted towards strong El Niño years, lowering the median number of WFD visits. This is the
**reference notebook set for footprint work**: `Footprint.ipynb` defines the per-pixel `NVisits`
(`CountMetric`/`HealpixSlicer`), reproduces the `fOBatch` "FP Comparison" figures, and compares the
baseline footprint against the `shrink_fp_dust_*` footprint variants. `07_variateEVmV` reuses these
cached results directly.

`Footprint.ipynb` is a local adaptation of the official Footprint Task Force notebook:
- Official version: [`lsst-pst/survey_strategy/fbs_5.3/Footprint.ipynb`](https://github.com/lsst-pst/survey_strategy/tree/main/fbs_5.3/Footprint.ipynb)
- This repository's version: [`03_fbs5.3.6/Footprint.ipynb`](https://github.com/sylvielsstfr/RubinLSSTOpsimAna/blob/main/notebooks/03_fbs5.3.6/Footprint.ipynb)

### `04_fbs5.3_SN`
`SNNSNMetric` (the DESC "number of SNe Ia" cadence metric) run on `baseline_v5.3.5_10yrs`, including the
reduced `n_sn` and `zlim` variants.

### `05_rubinsimutil_SN`
Exploration of the lower-level `rubin_sim` SN utility modules (`sn_utils`, `sn_n_sn_utils`) that
`SNNSNMetric` itself relies on internally.

### `06_MAF_DESC_TaskF`
Restricted to the DESC metrics relevant to the SCOC Task Force (see below): the **3x2pt** static-probes
Figure of Merit (galaxy clustering + weak lensing), **Weak Lensing** systematics-mitigation proxy metrics,
**Supernovae Ia** counts (`SNNSNMetric`), **Kilonovae** detection efficiency, and **Astrometry**
(parallax/proper motion/peculiar-velocity precision). Each science case has its own numbered notebook,
following a common convention (`data_<TAG>/` for MAF outputs, `figs_<TAG>/` for figures, saved as both
PNG and PDF). Full detail in `notebooks/06_MAF_DESC_TaskF/README.md`.

### `07_variateEVmV`
Directly supports the SCOC **Footprint Task Force** (see below): quantifies how moving the dust-extinction
cutoff E(B-V) used to define the WFD footprint boundary trades survey area against visits/depth, using
Healpix `NVisits`/`fONv` maps for seven `shrink_fp_dust_*` variants of `baseline_v5.3.6`. Full detail in
`notebooks/07_variateEVmV/README.md`.

### `08_imagequality`
Image-quality characterization of the cadence simulations, complementing the footprint/depth work of
`03_fbs5.3.6` and `07_variateEVmV` with the *delivered seeing* instead of visit counts or depth.
`01_Seeing_HealpixMaps.ipynb` builds Healpix maps (`nside=64`) and all-sky histograms of the median
per-visit `seeingFwhmEff` and `seeingFwhmGeom` (`MedianMetric` + `HealpixSlicer`) for
`baseline_v5.3.6_10yrs` and an earlier baseline (`baseline_v5.3.5_10yrs` if available), with a side-by-side
mosaic, a difference map, overlaid histograms, a per-band (`ugrizy`) breakdown and a summary table. It also
correlates the per-pixel airmass with the seeing maps, to check how much of the seeing differences between
runs comes from the airmass distribution of the scheduled visits. Full detail in
`notebooks/08_imagequality/README.md`.

### `09_RubinMaps`
Inspection of the HEALPix maps shipped in the `maps` data directory of `rubin_sim` (used by the MAF `maps`
classes; a local copy lives in `09_RubinMaps/maps`, fallback `~/DATA/OpSim/maps`):
- `01_DustMaps.ipynb` — SFD (1998) E(B-V) maps (HEALPix NSIDE 2-1024 and the original Lambert FITS maps):
  full-sky views, E(B-V) distribution and versus Galactic latitude, effect of the resolution.
- `02_DustMaps3d.ipynb` — 3D dust maps (E(B-V) versus distance per pixel): lines of sight, maps at fixed
  distance, comparison of the map variants and with the 2D SFD map.
- `03_mapsStellarDensity.ipynb` — cumulative stellar density maps (NSIDE 64, `ugrizy` and white dwarfs):
  dependence on band and limiting magnitude, Galactic latitude, counts N(<m) along selected lines of sight.

The TRILEGAL maps (`TriMaps`) and the Galactic-plane priority maps are not examined yet. Full detail in
`notebooks/09_RubinMaps/README.md`.

### `10_DESCMAFDEPTHANDGALCOUNT`
Detailed study of the DESC metrics of `rubin_sim.maf`, **recomputed on the 7 cadence simulations whose WFD
footprint is defined by a dust threshold** (`shrink_fp_dust_{0.050 ... 0.250}_v5.3.6` and the baseline), with
the E(B-V) cut of each metric adapted to each run. The aim is to understand how these metrics are built and
where they differ, in order to improve them later (PSF handling, weak-lensing metric). For each metric: Healpix
maps on a common color scale, area-weighted histograms, and difference maps between consecutive thresholds.
Metrics covered:
- effective surface area (`ExgalM5WithCuts` + `CountRatioMetric`; custom `n_eff`-weighted version),
  `ExgalM5WithCuts`, `GalaxyCountsMetricExtended` vs `DepthLimitedNumGalMetric`,
- weak-lensing proxies (`WeakLensingNvisits`, `RIZDetectionCoaddExposureTime`) and a seeing/depth-dependent
  `n_eff` model (Chang et al. 2013), first step towards a seeing-aware WL metric,
- `StaticProbesFoMEmulatorMetric` (3x2pt FoM, year by year), `SNNSNMetric` (`n_sn`, `zlim`), `KNePopMetric`,
  tomographic sigma8 bias, and `TdcMetric` (time-delay cosmography),
- `100_MAFDESCComparison_demo.ipynb` reads the caches of all the others and draws every `MAF / MAF(baseline)`
  curve versus the E(B-V) threshold on one plot.

Full detail in `notebooks/10_DESCMAFDEPTHANDGALCOUNT/README.md`.

### `10_DESCMAFDEPTHANDGALCOUNT_FootpTF`
Footprint-Task-Force version of the previous series (no README of its own yet): same notebooks
`00` to `10` and `100` (the TDC notebook `11` and the `00b`/`DOC00` notebooks are not duplicated), with the
set of cadences extended from 7 to **10 runs** by three additional E(B-V) thresholds (0.090, 0.100 and 0.110)
and the plotting functions adapted accordingly. It also contains two variants of the 3x2pt FoM notebook `06`
(`..._offsetminus05` and `..._offsetminus09`, with their own `data_06_FOM3X2PTS_offstm05/09` and
`figs_06_FOM3X2PTS_offstm05/09` outputs). Same `data_<NN_TAG>/` / `figs_<NN_TAG>/` convention.

### `11_NEWDESCMAF`
Work in progress (draft, not yet executed): proposals for new DESC MAF metrics **without modifying
`rubin_sim`**. `00_DustLimitedNumGalMetric.ipynb` defines `DustLimitedNumGalMetric`, which follows the logic of
`DepthLimitedNumGalMetric` (built on `GalaxyCountsMetricExtended`) but with an easily variable `lim_ebv`, so that
the hard E(B-V) footprint cut is separated from the continuous loss of depth due to extinction; and
`EBVAreaMetric`, the angular area satisfying E(B-V) < `lim_ebv`. Combined, they give the total number of
galaxies N_gal(E_max), the retained area Omega(E_max) and the mean density n_gal = N_gal / Omega as functions of
the E(B-V) cut.

### `12_SurveyStrategy`
Characterization of the LSST survey strategy in `baseline_v5.3.6_10yrs`, adapted (with added comments) from the
official notebooks of [`survey-strategy_lsst_io`](https://github.com/lsst/survey-strategy_lsst_io), with the
generated files sorted into `DATANN_<Name>/` (tables, MAF arrays) and `FIGNN_<Name>/` (figures). The survey modes
are WFD regions (low dust, bulge, Virgo, SMC-LMC), mini-surveys (NES, SCP, Milky Way non-WFD) and micro-surveys
(RGES, near-sun twilight, ToO). Notebooks:
- `01_SurveyModes`, `02_SurveyAreas` — goal footprint, area and number of visits of each survey mode,
- `03_N_Per_Season`, `06_DDF_RollingUniformity` — visits per season and survey uniformity as epochs accumulate
  (rolling cadence),
- `04_DDF_NVisitsVsTime`, `05_DDF_DepthVsTime` — cumulative visits and coadded depth of each Deep Drilling Field,
- `07_WFDMiniMicroS_NVisitsVsTime`, `09_WFDLowDust_DepthVsTime` — the same timelines for the WFD, mini- and
  micro-surveys, and depth of the low-dust WFD compared with the DDFs,
- `08_Footprint_TaskForce` — current versus modified footprint (stellar density and E(B-V) cut), independent of
  the OpSim database,
- `10_FilterBalance` — filter balance (ugrizy) versus time for each survey type,
- `13_Multiplets` — singlets, doublets, triplets... of visits versus survey mode and sky position.

Full detail in `notebooks/12_SurveyStrategy/README.md`.

## The SCOC Footprint Task Force

The notebooks in `03_fbs5.3.6`, `06_MAF_DESC_TaskF`, `07_variateEVmV`, `10_DESCMAFDEPTHANDGALCOUNT` (and its
`_FootpTF` extension) and `12_SurveyStrategy/08_Footprint_TaskForce` all feed into the same piece of
work: the Rubin/LSST **Survey Cadence Optimization Committee (SCOC) Footprint Task Force**.

Successive updates to the survey forecast (weather years, downtime, observatory performance) reduced the
expected total number of visits over the 10-year survey from roughly 2M+ down to about 1.8M. Between the
`v5.0` and `v5.3.6` baselines this cut the median number of visits per point in the Wide-Fast-Deep (WFD)
survey from about 770 down to about 660 — below the SRD design goal of 825 (and getting close to the SRD
*minimum* of 750, over the design area of 18,000 sq deg / minimum area of 15,000 sq deg, per the LSST
Science Requirements Document, [ls.st/srd](https://ls.st/srd)).

The Footprint Task Force is evaluating options to bring the survey back within the SRD requirements by
redistributing visits within the current footprint and/or **shrinking the WFD area** — from its current
~20,000 sq deg down to somewhere between 15,000 and 18,000 sq deg — which raises the number of visits per
point in the remaining WFD area. The main lever explored in the preliminary simulations is the
dust-extinction cutoff E(B-V) used to define the boundary of the low-dust WFD region: tightening it
(e.g. from 0.2 down to 0.08) shrinks the WFD footprint near the Galactic plane and converts the freed
visits into extra depth over the remaining, still-large extragalactic footprint. The task force also
tracks the effect of this trade-off on downstream science metrics (DESC 3x2pt Figure of Merit and
effective survey area, supernovae, kilonovae, microlensing, and other time-domain science).

In this repository:
- `03_fbs5.3.6/Footprint.ipynb` reproduces the official `fOBatch`-based footprint comparison figures.
- `07_variateEVmV/` quantifies, pixel by pixel, the effect of moving the E(B-V) cutoff across the full set
  of `shrink_fp_dust_*` simulations available for `v5.3.6`.
- `06_MAF_DESC_TaskF/` tracks the DESC-specific science metrics (3x2pt, WL, SN, KNe, astrometry) that the
  task force uses to judge whether a given footprint choice preserves cosmology and time-domain science.
- `10_DESCMAFDEPTHANDGALCOUNT/` (and `10_DESCMAFDEPTHANDGALCOUNT_FootpTF/`, with 10 thresholds) recomputes the
  DESC metrics (depth, galaxy counts, WL, 3x2pt FoM, SN, KNe, sigma8, time delays) for each E(B-V) threshold and
  compares them to the baseline.
- `12_SurveyStrategy/08_Footprint_TaskForce.ipynb` builds and displays a modified footprint (joint cut on stellar
  density and E(B-V)) next to the current one, with the area of each region.

## Installation

### 1. Install `rubin_sim`

`rubin_sim` is listed as a dependency in `pyproject.toml`, so it is installed automatically together with
this package (see below). Standalone, the two supported routes are:

```bash
# Conda (recommended if you are not modifying rubin_sim itself)
conda create -n rubin-sim -c conda-forge rubin_sim
conda activate rubin-sim

# or pip
pip install rubin-sim
```

To install this package itself for local development:

```bash
conda create -n <env_name> python=3.11
conda activate <env_name>
./.setup_dev.sh          # editable install + dev dependencies + pre-commit
conda install pandoc     # optional, needed to render notebooks into the docs
```

### 2. Download the auxiliary `rubin_sim` data

`rubin_sim`/MAF needs an auxiliary data directory (throughputs, dust maps, moving-object SEDs,
pre-computed sky brightness, etc.) that is *not* shipped with the pip/conda package:

```bash
export RUBIN_SIM_DATA_DIR=$HOME/rubin_sim_data   # optional, defaults to ~/rubin_sim_data
rs_download_data                                 # downloads a few GB of auxiliary data
rs_download_data --version                       # check which data version is installed
rs_download_data --force --dirs throughputs,skybrightness,skybrightness_pre   # update specific dirs
```

### 3. Download the OpSim simulation databases

The actual survey-cadence simulation outputs (the `.db` files analyzed by the notebooks), and the
pre-computed MAF metrics for them, are a separate download from the S3DF data portal. Recommended
lookup order:

1. **Start with the run/MAF summary table**: [`https://usdf-maf.slac.stanford.edu/`](https://usdf-maf.slac.stanford.edu/)
   lists all official runs together with their pre-computed MAF metrics, and is the fastest way to find
   a given run and its `summary.h5` file (the master table of MAF summary-metric values for every run,
   used e.g. by `03_fbs5.3.6/Footprint.ipynb`'s FoM comparisons):
   `https://s3df.slac.stanford.edu/data/rubin/sim-data/sims_featureScheduler_runs5.3/maf/summary.h5`
2. **If a run is not listed there**, fall back to the raw simulation directories:
   - v5.3 baselines: `https://s3df.slac.stanford.edu/data/rubin/sim-data/sims_featureScheduler_runs5.3/baseline/`
   - v5.3.6 footprint-task-force variants: `https://s3df.slac.stanford.edu/data/rubin/sim-data/sims_featureScheduler_runs5.3/shrink_fp/`

By convention in this repository, downloaded `.db` files are kept under `/Users/dagoret/DATA/OpSim/`
(the baseline under `sim_baseline/`), which is what the notebooks point to.

**Files needed specifically for the SCOC Footprint Task Force** (`03_fbs5.3.6`, `06_MAF_DESC_TaskF`,
`07_variateEVmV`):

| File | Role |
|---|---|
| `sim_baseline/baseline_v5.3.6_10yrs.db` | Current official baseline (E(B-V) cut ≈ 0.20); used by `03_fbs5.3.6` and `06_MAF_DESC_TaskF` |
| `shrink_fp_dust_0.050_v5.3.6_10yrs.db` | Footprint shrunk to E(B-V) ≤ 0.050 |
| `shrink_fp_dust_0.080_v5.3.6_10yrs.db` | Footprint shrunk to E(B-V) ≤ 0.080 — meets the SRD minimum |
| `shrink_fp_dust_0.120_v5.3.6_10yrs.db` | Footprint shrunk to E(B-V) ≤ 0.120 — close to, but just under, the SRD minimum |
| `shrink_fp_dust_0.150_v5.3.6_10yrs.db` | Footprint shrunk to E(B-V) ≤ 0.150 |
| `shrink_fp_dust_0.199_v5.3.6_10yrs.db` | Footprint shrunk to E(B-V) ≤ 0.199 (≈ baseline cut) |
| `shrink_fp_dust_0.250_v5.3.6_10yrs.db` | Footprint grown to E(B-V) ≤ 0.250 |
| `sim_baseline/baseline_v5.3.0_10yrs.db` | Pre-v5.3.6 baseline, used as a reference point in `03_fbs5.3` comparisons |

All seven `shrink_fp_dust_*` runs plus the `baseline_v5.3.6_10yrs` run are used together in
`07_variateEVmV/01_FOMNv_HealpixDiff_ShrinkFPDust.ipynb`.
