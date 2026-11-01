# 11_SCOC_FootPTF_Final: LSST footprint task force notebooks

This folder collects the notebooks of the **SCOC Footprint Task Force** (chair: Knut Olsen), imported from
<https://github.com/knutago/footprint-taskforce> and run locally with the `conda_py313_opsim53` kernel.
They build and tune a revised LSST survey footprint for a survey of roughly **1.8 to 2 million visits**
instead of the 2.2 to 2.6 million assumed in earlier simulations.

The reasoning, the science considerations and the resulting scenarios are described in the task force report:

- `report/Footprint-task-force_report.pdf` (also readable at
  <https://docs.google.com/document/d/1IsbjRHAwiLU4EK5MgLIEhm_h805BcHE7I6SRFzALkx0>)

## Context (from the report)

- Simulations updated with the real system performance, downtime and weather forecast give a median of about
  **660 visits per WFD pointing**, below the SRD minimum of 750 (the Big Sky white paper assumed about 825).
- The reduced number of visits hurts the time-domain science pillar the most.
- A simple cut on E(B-V) alone shrinks the extragalactic WFD, which hurts area-driven metrics (number of galaxies,
  3x2pt Dark Energy figure of merit, brown dwarfs) and the solar-system metrics near the ecliptic.

## What the proposed footprint contains

1. **Extragalactic low-dust WFD (`lowdust`)**: defined by a *joint cut* on stellar density and E(B-V), instead of
   an E(B-V) cut alone (`STARDENS_CUT`, `EBV_CUT`).
2. **Galactic Plane WFD (`rgps`)**: replaces the baseline `bulgy` region. It follows the TVS priority map
   (Street et al. 2023) and the Roman Galactic Plane Survey footprint, with a longitude limit
   (`RGPS_NORTH_L_MIN`) and no small island fields.
3. **Bridge (`bridge`)**: a band along the ecliptic, observed at WFD level, that crosses the Galactic Plane and
   connects the two parts of the extragalactic footprint. It is useful for faint minor-planet discovery and for
   following stellar populations through a continuous range of stellar density.
4. The remaining area is observed at reduced levels: `dusty_plane` (170 visits per pointing), North Ecliptic Spur
   `nes` (150) and South Celestial Pole `scp` (100). WFD-level regions get 750 visits per pointing.

Scenarios given in the report:

| Scenario | Total visits | `lowdust` cut (E(B-V) and star density) | Galactic Plane WFD | Bridge width |
|---|---|---|---|---|
| 1 | about 1.95 million | E(B-V) < 0.12, n(r<17) < 2e-4 arcsec^-2 | about 1,500 deg^2 | about 14 deg |
| 2 | 1.8 million | E(B-V) < 0.12, n(r<17) < 9e-5 arcsec^-2 (about 13,000 deg^2) | about 1,380 deg^2 | about 8 deg |

The three knobs proposed in the report to tune the visit budget are `STARDENS_CUT`, `RGPS_EDGE_TRIM_FRAC` and
`BRIDGE_TARGET_AREA`.

## Notebooks

All notebooks use `rubin_sim` MAF, `rubin_scheduler`, `healpy` and `astropy`. They work at HEALPix `NSIDE = 64`,
and do not need an OpSim database, except in the optional visit-counting step of the TVS region (see below).

| Notebook | Purpose |
|---|---|
| `01_footprint_task_force.ipynb` | Starting point. Computes the stellar density (TRILEGAL or GalFast) and E(B-V) maps on a HEALPix grid, applies the joint cut, and plots it against the current baseline footprint and WFD. Compares the cut with an external map of the usable WFD fraction. Builds a first modified footprint: the joint cut replaces `lowdust`, the edge is smoothed, holes are filled by non-WFD regions, small `bulgy` islands are removed and a Galactic-coordinate box "bridge" joins `bulgy` and `lowdust`. Reports areas, pointings and visits per component. |
| `02_footprint-taskforce_GPmod_1.ipynb` | Intermediate version with the Galactic Plane modification. Adds the TVS SC proposed Galactic-plane region (priority map plus Roman RGPS boxes) and builds the modified footprint with the new Galactic Plane region and an ecliptic bridge. Exports the result as a HEALPix FITS file and as a scheduler-style `.npz`. |
| `03_footprint-taskforce_refactor.ipynb` | Refactored version, which is the one behind **Scenario 1**. All parameters are in one Settings cell, and the footprint construction is wrapped in `build_footprint(stardens_cut, edge_trim_frac, bridge_target_area)`. Optionally solves `STARDENS_CUT` for a target number of visits (`TARGET_TOTAL_VISITS`). Produces the area and visit tables, the map of the modified footprint (with the Roman and DECaPS2 footprints and the ecliptic) and the exported maps. |
| `04_footprint-taskforce_refactor_1p8M.ipynb` | Same code as notebook 03, tuned for **Scenario 2** (`TARGET_TOTAL_VISITS = 1.8e6`, `RGPS_EDGE_TRIM_FRAC = 0.10`, `BRIDGE_TARGET_AREA = 200`). Output files carry the `1p8M` tag. |

### Typical workflow

1. Run the environment check (section 1) and make sure the `rubin_sim` data are available (`$RUBIN_SIM_DATA_DIR`).
   If needed, run `rs_download_data --dirs maps` and `scheduler_download_data`.
2. Edit the Settings cell (cuts, region budgets, visits per region).
3. Run all cells. In notebooks 02 to 04, the TVS priority map is downloaded from GitHub, and the Roman and DECaPS2
   footprints are fetched online when available (with a fallback to published boxes).
4. For a new visit budget, change `STARDENS_CUT`, `RGPS_EDGE_TRIM_FRAC` or `BRIDGE_TARGET_AREA`, or set
   `TARGET_TOTAL_VISITS` and let notebooks 03 and 04 solve the stellar-density cut.

## Input files

- `rubin_wfd_stellar_masked_nside64.fits`: external HEALPix map (column `wfd_usable_fraction`, the usable fraction
  of each WFD pixel after stellar masking), used for the comparison in section 9 of notebooks 01 and 02.
- The TVS priority map is read from the `LSST-TVSSC/software_tools` repository.
- Optionally, the baseline OpSim database (`baseline_v5.3.0_10yrs.db`, `TVS_OPSIM_DB`) is used to count visits
  per pixel. Without it, the notebook falls back to a uniform number of visits per pixel.

## Outputs

Each notebook writes its outputs in two separate folders, created at the first run. `XX` is the notebook number.

- `figs_XX/`: all figures (PNG), such as the stellar-density map with E(B-V) contours and the joint cut, the
  comparison with the external map, and the map of the modified footprint.
- `data_XX/`: all data products:
  - area and visit tables (`*_areas.csv`, `*_visits.csv`),
  - the modified footprint as an integer label map (`footprint_labels_cut_*.fits`, notebooks 01 and 02),
  - the exported footprint (`footprint_GPmod_*nside64.fits` with the columns `label`, `wfd`, `median_visits`,
    `u` to `y`, and the matching `.npz`, notebooks 02 to 04),
  - the intermediate MAF outputs (`tvs_maf_out/`).

## Credits

The notebooks and the footprint proposal are the work of the Footprint Task Force (K. Olsen, chair; E. Bellm,
S. Dagoret-Campagne, M. Dominguez, J. Freeburn, L. Jones, M. Knight, R. Mandelbaum, P. McGehee, R. Street,
K. Vivas, P. Yoachim). The Galactic Plane region follows Street et al. (2023, ApJS, 267, 15), and the TVS
construction follows `GalPlaneSurvey/modified_galplane_footprint.ipynb` from `LSST-TVSSC/software_tools`.

## Caveats

- The report limits itself to the survey footprint: Deep Drilling Fields, Target-of-Opportunity follow-up and
  filter balance are not considered.
- The 3x2pt metric does not account for the stellar-density-dependent masking, so the area-driven metrics are
  slightly optimistic. Conversely, the baseline 5.3 estimate of the usable area was overly pessimistic.
