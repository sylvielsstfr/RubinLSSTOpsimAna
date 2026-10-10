# 12_SurveyStrategy

Notebooks that characterise the **LSST survey strategy** in the Rubin baseline simulation `baseline_v5.3.6_10yrs.db` (OpSim), using the scheduler target footprint (`rubin_scheduler`) and the MAF framework (`rubin_sim`). They are adapted from the official notebooks of https://github.com/lsst/survey-strategy_lsst_io, with added comments, and with the generated files sorted into separate directories.

Kernel: `conda_py313_opsim53`.

## Conventions

- Each notebook `NN_Name.ipynb` writes its **tables and MAF arrays** to `DATANN_Name/` and its **figures** to `FIGNN_Name/`. The directories are created by the notebook if they do not exist.
- The OpSim database path is set in one variable at the top of each notebook (`opsdb` or `opsim_fname`). Change it to analyse another run.
- All comments and documentation are in English.
- Time is expressed in years since the survey start (`SURVEY_START_MJD`, 1 year = 365.25 d) in the timeline notebooks.

## Survey modes

| Mode | Content |
|---|---|
| WFD regions | low dust (`lowdust` + `euclid_overlap`), bulge (`bulgy`), Virgo, SMC-LMC (`LMC_SMC`) |
| Mini-surveys | North Ecliptic Spur (`nes`), South Celestial Pole (`scp`), Milky Way non-WFD (`dusty_plane`) |
| Micro-surveys | DDF, RGES, near-sun twilight, ToO, (third visits of triplets are WFD visits) |

## Notebooks

### 01_SurveyModes

Describes the **on-sky goal footprint** of each survey mode and the typical visits per band in each area.

1. Plots the scheduler target map by survey mode (WFD regions, mini-surveys).
2. Lists the DDF sky positions (equatorial, galactic, ecliptic coordinates).
3. Counts the visits per `scheduler_note` (pairs, triplets, DDF, twilight, ...).
4. Runs MAF metrics per region: number of visits, coadded depth, median single-visit depth.
5. Maps the near-sun twilight micro-survey and its solar-elongation distribution.

Outputs: `DATA01_SurveyModes/` (MAF `.npz`), `FIG01_SurveyModes/`.

### 02_SurveyAreas

Measures the **sky area and number of visits of each survey mode** (WFD, mini-surveys, DDF, twilight, ToO, ...).

1. Shows the dust map and the label map of the target footprint, and the area of each label.
2. Counts visits and plots visit maps for all visits, DDF, RGES, near-sun twilight and triplet third visits.
3. Attributes visits to regions with `WFDlabelStacker` (a visit counts for a region if more than 40% of its field of view lies in it) and maps the visits of each region.
4. Computes per-band visit maps and the median/mean number of visits inside each region.

Outputs: `DATA02_SurveyAreas/` (MAF `.npz`), `FIG02_SurveyAreas/`. The figures of this notebook are mostly displayed only (MAF `plot()` does not save them).

### 03_N_Per_Season

Studies the **number of visits per observing season** at each position of the WFD footprint, to characterise the **rolling cadence**.

1. Custom MAF metric `NVisPerSeasonMetric`: number of visits, maximum and median gap between nights, season length.
2. Runs it on the WFD pixels.
3. Classifies each season at each position as *on* (many visits), *off* (few visits) or *average*, and compares the cadence of the three groups.
4. Maps the number of visits per season.

Outputs: `DATA03_N_Per_Season/`, `FIG03_N_Per_Season/`.

### 04_DDF_NVisitsVsTime

**Timeline of the Deep Drilling Fields** (Ocean DDF strategy: each DDF alternates shallow and deep seasons, so the cumulative curves are staircases).

1. Loads the DDF visits (`scheduler_note` containing `DD`, RGES excluded) and assigns each to a DDF.
2. Summary table per DDF: visits, fraction of the survey, nights, first/last visit, position, visits per band, exposure time.
3. Figures:
   1. cumulative visits per DDF versus time;
   2. cumulative visits per DDF and per band;
   3. visits per month and per DDF (heat map);
   4. nights on which each DDF is observed (timeline);
   5. visits per DDF night and gaps between DDF nights;
   6. cumulative fraction of the survey visits spent in DDFs.

Outputs: summary table in `DATA04_DDF_NVisitsVsTime/`, figures in `FIG04_DDF_NVisitsVsTime/`.

### 05_DDF_DepthVsTime

**Cumulative 5-sigma depth versus time for each DDF**, per LSST band, as a function of the calendar date (depth counterpart of notebook 04).

1. Loads the DDF visits from the OpSim database (COSMOS, EDFS, XMM-LSS, ELAIS-S1, ECDFS; the two EDFS pointings are merged).
2. Computes, for each DDF and each band, the coadded depth after each visit: `m5_cum = 1.25 log10(sum 10**(0.8 m5_i))`.
3. Summary tables per DDF and band: number of visits, median single-visit depth, final coadded depth and depth gain.
4. Figure 1: cumulative depth versus date, one panel per DDF, one curve per band, survey years marked.

Outputs: tables and the cumulative depth of every visit (`.csv`) in `DATA05_DDF_DepthVsTime/`, figure (PNG and PDF) in `FIG05_DDF_DepthVsTime/`.

### 06_DDF_RollingUniformity

Tests the **uniformity of the WFD (without DDF) as epochs accumulate**, which matters for static cosmology (large-scale structure, weak lensing, 3x2pt). Based on `03_N_Per_Season` (same footprint and SQL constraint).

1. Custom MAF metric `CumulativeVisitsMetric`: cumulative number of visits and cumulative coadded depth in one band, for N = 1 ... 10, with two definitions of "after N epochs": **time-based** (visits before `t0 + N years`) and **season-based** (visits in the first N seasons of each sky position, which removes the RA-dependent season offset).
2. Runs it on the WFD pixels.
3. Maps the cumulative number of visits for each epoch (absolute and normalised to the median).
4. Quantifies the uniformity versus epoch (relative RMS, robust sigma, 5-95 percentile spread, fraction of pixels within 10% of the median) and compares the two definitions.
5. Same for the coadded depth in one band.

Outputs: arrays and tables in `DATA06_DDF_RollingUniformity/`, figures (PNG and PDF) in `FIG06_DDF_RollingUniformity/`.

### 07_WFDMiniMicroS_NVisitsVsTime

Counterpart of notebook 04 for everything that is **not a single DDF**: cumulative number of visits versus time for each WFD region, each mini-survey and each micro-survey.

1. Builds the region labels from the scheduler target map and attributes each visit to **one** category: micro-surveys from `scheduler_note`, other visits from the label of the HEALPix pixel (nside 64) containing the visit centre. The three totals add up to the number of visits of the survey.
2. Summary table per region (visits, share of the survey, nights, first/last visit, area, exposure time, visits per band) and visits per survey year.
3. Figures:
   1. cumulative visits in the WFD regions (low dust, bulge, Virgo, SMC-LMC) and total WFD;
   2. cumulative visits in the mini-surveys and total mini-surveys;
   3. cumulative visits in the micro-surveys and total micro-surveys (third visits of triplets drawn as an overlay);
   4. overview: total WFD, total mini-surveys, total micro-surveys and all visits;
   5. fraction of the survey visits versus the total number of visits, with the sum of all DDFs;
   6. same fraction versus time.
   (Figures 1-4 have a linear and a log panel.)
4. Final summary table for the main regions (DDF, WFD, mini-surveys, micro-surveys without DDF, others): area in deg2, fraction of the area, number of visits and fraction of the visits after 10 years. Areas of WFD and mini-surveys come from the target map; areas of DDF and micro-surveys are the sky covered by the field of view of their visits, so areas of different rows overlap.

Outputs: tables and cumulative curves (`.csv`) in `DATA07_WFDMiniMicroS_NVisitsVsTime/`, figures (PNG and PDF) in `FIG07_WFDMiniMicroS_NVisitsVsTime/`.

### 08_Footprint_TaskForce

Proposal for a **modified survey footprint**, adapted from the footprint task force notebook https://github.com/knutago/footprint-taskforce (`footprint_task_force.ipynb`). It does not need an OpSim database: it uses the star-density map (TRILEGAL) and the SFD dust map of `rubin_sim` together with the scheduler footprint (`get_current_footprint`).

1. Loads `TrilegalDensityMap` and `DustMap` on a full-sky HEALPix grid (nside 64), evaluates `StarDensityMetric` (stars brighter than r = 17) and checks the coordinate frame of each map.
2. Reads the current footprint labels (`lowdust`, `euclid_overlap`, `virgo`, `bulgy`, `LMC_SMC`, `nes`, `dusty_plane`, `scp`) and the WFD area.
3. Applies a **joint cut** on stellar density and E(B-V) and plots the stellar density with E(B-V) contours, the pixels passing the cut, the WFD and the full footprint.
4. Builds the **modified footprint** in five steps: cut of `lowdust`, smoothing of its edge, filling of the holes with the nearest region, removal of small `bulgy` islands and trimming of the `bulgy` edge, and a `bridge` between `bulgy` and `lowdust`.
5. Tables of area, pointings and visit budget per component, for the current and the modified footprint.
6. Sky maps of the **current footprint** and of the **modified footprint**, with the same colour for the same region and the area (deg2) of each region in the legend; the modified map also shows the outline of the current `lowdust` and the ecliptic. On both maps the WFD regions are outlined in red, the mini-surveys (`nes`, `scp`, `dusty_plane`) in blue, and the DDF fields (enlarged circles, white fill with a solid magenta edge, name written above each field).

Outputs: tables (`.csv` of areas and visits) and the label map of the modified footprint (`.fits`) in `DATA08_Footprint_TaskForce/`, figures (PNG and PDF) in `FIG08_Footprint_TaskForce/`.

## Reading order

`01_SurveyModes` and `02_SurveyAreas` define the survey modes and their areas. `03_N_Per_Season` and `06_DDF_RollingUniformity` study the rolling cadence of the WFD. `04`, `05` and `07` follow the visits and depth of the DDFs, the WFD regions, the mini-surveys and the micro-surveys as a function of time. `08_Footprint_TaskForce` is independent of the OpSim database and compares the current footprint with a modified one.
