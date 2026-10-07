# Xenolith melt vein modeling

Olivine diffusion modeling for:

Towbin, W. H., and Plank, T. (2026). Decompression Melting of Peridotite Xenoliths. *Journal of Petrology* 67(9), egag077. https://doi.org/10.1093/petrology/egag077

Spinel lherzolite xenoliths from Sullivan Ranch, a cinder cone north of the Grand Canyon, are cut by melt veins. Primary olivine next to the veins is zoned in Fe-Mg. Fitting diffusion models to those zoning profiles gives how long the melting lasted: 1-3 weeks in the paper, similar to the roughly one week recorded by olivine at the xenolith exteriors that was equilibrating with the host magma.

This is the code as used for the paper, taken from commit 7c6bf5f of [Olivine_Diffusion_FeMg](https://github.com/whtowbin/Olivine_Diffusion_FeMg) (tag `paper-jpet-2026`, February 2026). Later development continues in that repo.

## What the script does

`Xenolith_Melt_Veins_Diffusion_Modeling_10_2025.py` runs top to bottom (it is also split into `#%%` cells for interactive use):

1. Loads the electron microprobe olivine profiles and a parameter table with one row per profile (EBSD orientation, profile angle, model edges and initial step, temperature, pressure, fO2, grid spacing and time step).
2. For each of the 18 profiles, interpolates the measured Fo profile onto an even grid by ordinary kriging, sets up a step-function initial condition, runs 1-D Fe-Mg diffusion forward in time with a diffusivity that depends on temperature, pressure, fO2, olivine composition and crystal orientation, and picks the best-fit time and its uncertainty by chi-squared. Each fit is saved as an SVG in `Xenolith_Ol_Diffusion_Models/`.
3. Rescales the best-fit times for a decompression path: it samples 10,000 pressure-temperature-fO2 paths from the xenolith entrainment depth to the surface and divides each time by the mean diffusivity along the path. Results go to `Times_database_Diffusivities.xlsx`.
4. Summarizes the rescaled times (median and 5th-95th percentiles) for each profile and for each xenolith in `Diffusion_Timescales_Summary.xlsx`, and plots them in `Melt_Vein_Olivine_Mg-Fe_Diffusion_Timescales_sum.svg`.

The other two files are imported by the main script:

- `Fe_Mg_Diffusion_Convolution_Streamlined.py`: the diffusion engine (diffusivity laws for Fe-Mg, Ni and Ca, fO2 buffers, the finite-difference time stepper, kriging interpolation and the R² and chi-squared fit scores).
- `EBSD_Profile_Rotation.py`: converts EBSD Euler angles and the profile direction into the angles between the profile and the olivine crystal axes, which set the diffusion anisotropy.

## Data

- `Xenolith_MeltVein_EMPA_Master.xlsx`: electron microprobe analyses of olivine (the `Olivine` sheet of the original working spreadsheet, saved as values only).
- `Olivine_Profile_Diffusion_Modeling_Parameters_DB.xlsx`: model settings per profile. The `units` sheet gives the units of each column.

The full data set for the paper, including the other phases, is in the paper's supplementary material and the EarthChem Library: https://doi.org/10.60520/IEDA/114314

## Running it

Requires [uv](https://docs.astral.sh/uv/) and Python 3.13 (uv installs it if needed).

```bash
git clone https://github.com/whtowbin/Xenolith_Melt_Vein_Modeling
cd Xenolith_Melt_Vein_Modeling
uv sync
MPLBACKEND=Agg uv run python Xenolith_Melt_Veins_Diffusion_Modeling_10_2025.py
```

A run takes about two minutes. `MPLBACKEND=Agg` saves figures without opening windows; without it, `EBSD_Profile_Rotation.py` opens a plot window on import and the script waits until it is closed. The figures use the Avenir font when it is installed and fall back to the matplotlib default otherwise.

The best-fit times and their uncertainties are deterministic and match the committed results exactly. The decompression rescaling is a Monte Carlo calculation with no fixed random seed, so the rescaled times and the summary change from run to run: by 1-2% for single profiles, and by up to about 15% for the pooled median of xenolith WHT06, whose profiles record very different times (about 1 and 22 days).

## Changes from the paper version

One line differs from commit 7c6bf5f: the call `g.vector_direction(...)` in `model_diffusion` now reads `EBSD.vector_direction(...)`. `g` was a name left over from an interactive session and is not defined in the script, so the original stopped with a `NameError` when run from a fresh start. The function and its results are unchanged.

## Citation

Please cite the paper above if you use this code. `CITATION.cff` has the details.

## License

MIT. See [LICENSE](LICENSE).
