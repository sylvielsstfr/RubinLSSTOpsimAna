# 09_RubinMaps

Inspection of the HEALPix maps shipped in the `maps` data directory of
[rubin_sim](https://github.com/lsst/rubin_sim) (used by the MAF `maps` classes).

Author: Sylvie Dagoret-Campagne
Kernel: `conda_py313_opsim53`

## Data

The `maps` directory was copied into this folder (`./maps`) so that the notebooks can read it directly.
The original location is `~/DATA/OpSim/maps`, which each notebook uses as a fallback.
The data are not meant to be versioned.

| Sub-directory | Content | Used by |
|---------------|---------|---------|
| `maps/DustMaps` | SFD E(B-V) maps as HEALPix (NSIDE 2 ... 1024) and original SFD Lambert FITS maps (NGP/SGP, 4096 x 4096) | `01_DustMaps.ipynb` |
| `maps/DustMaps3D` | 3D dust maps: E(B-V) versus distance for each HEALPix pixel (several variants of merged 3D maps) | `02_DustMaps3d.ipynb` |
| `maps/StarMaps` | Stellar density maps (NSIDE 64), cumulative in magnitude, for `ugrizy` and for white dwarfs only | `03_mapsStellarDensity.ipynb` |
| `maps/TriMaps` | TRILEGAL stellar density maps | not examined yet |
| `maps/GalacticPlanePriorityMaps` | Galactic plane priority maps | not examined yet |

See also `maps/README.md` (original README of the `sims_maps` data).

## Notebooks

### `01_DustMaps.ipynb` - Galactic dust maps (SFD 1998)

* Inventory of the available resolutions and basic statistics of the E(B-V) maps (key `ebvMap`, RING ordering, pixel centres on RA/Dec).
* Full-sky maps in equatorial and Galactic coordinates, views towards the Galactic centre and the poles.
* Distribution of E(B-V), cumulative sky fraction, E(B-V) versus Galactic latitude.
* Effect of the HEALPix resolution.
* Original SFD Lambert FITS maps and cross-check with the HEALPix maps.

### `02_DustMaps3d.ipynb` - 3D Galactic dust maps

* Headers and sanity checks of the large FITS files (opened memory-mapped, processed by blocks of pixels).
* E(B-V) as a function of distance along selected lines of sight.
* Full-sky maps at fixed distances.
* Far end of the 3D maps compared to the 2D SFD map.
* Distance reached for a given `m - M0`.
* Comparison of the map variants (`defaults`, `bridge`, `noL19`, `merged`) and effect of NSIDE (64 vs 128).

### `03_mapsStellarDensity.ipynb` - Stellar density maps

* Structure of the `starDensity_<band>[_wdstars]_nside_64.npz` files: `starDensity` `(npix, 65)` cumulative counts,
  `bins` (magnitude edges 15.0 - 28.0, step 0.2), `overMaxMask`.
* Helper `star_density(band, maglim, wd=False)` giving the density of stars brighter than `maglim`
  (linear interpolation between magnitude bins).
* Full-sky maps (equatorial and Galactic), dependence on the limiting magnitude and on the band.
* Views towards the Galactic centre and anticentre, density versus Galactic latitude.
* Cumulative counts N(<m) along a few lines of sight.
* White dwarf maps and their fraction of the total density.

**Caveat**: the unit of `starDensity` (stars / deg^2 versus stars / pixel) is assumed to be stars / deg^2 and is not
stated in the files. To be confirmed against the code that generated the maps.

## Requirements

`numpy`, `matplotlib`, `healpy`, `astropy`, `scipy`, `pandas`.

## Next steps

* Compare `StarMaps` with the TRILEGAL maps in `maps/TriMaps`.
* Examine `maps/GalacticPlanePriorityMaps`.
