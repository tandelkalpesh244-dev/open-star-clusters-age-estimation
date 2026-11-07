# Comparative Age Analysis of Open Star Clusters (Pleiades, Hyades, M67)

A Google Colab notebook for a dissertation-style study of open star clusters. It pulls cluster catalogs and Gaia DR3 photometry from public astronomy archives, builds Hertzsprung–Russell (H-R) / colour–magnitude diagrams for three benchmark clusters, and compares cluster ages using main-sequence turn-off (MSTO) positions.

> **Status: work in progress.** Several parts of the notebook currently run on **simulated data** or **hard-coded values** rather than real measurements. Each section below says which. Read [Data Provenance & Limitations](#data-provenance--limitations) before using any figure or number in a report.

---

## Table of Contents

- [Objectives](#objectives)
- [Clusters Studied](#clusters-studied)
- [Data Sources](#data-sources)
- [Notebook Walkthrough](#notebook-walkthrough)
- [Outputs](#outputs)
- [Data Provenance & Limitations](#data-provenance--limitations)
- [Getting Started](#getting-started)
- [Known Issues](#known-issues)
- [Suggested Next Steps](#suggested-next-steps)
- [Acknowledgements](#acknowledgements)

---

## Objectives

1. Survey large open-cluster catalogs and summarize cluster ages by age group.
2. Retrieve Gaia DR3 stars around three clusters of very different ages and select likely members.
3. Plot H-R diagrams (BP−RP colour vs absolute G magnitude) and locate the main-sequence turn-off.
4. Show how the turn-off moves down and to the red with increasing age.
5. Compare estimated ages with literature values and quantify uncertainties.

## Clusters Studied

| Cluster | Age (literature) | Distance | Notes |
|---|---|---|---|
| Pleiades | 0.125 Gyr | 136.2 pc | Young; hot, luminous upper main sequence |
| Hyades | 0.625 Gyr | 46.3 pc | Intermediate age; nearest open cluster |
| M67 | 4.0 Gyr | 908 pc | Old; red giants and a low turn-off |

## Data Sources

| Source | Used for | Access |
|---|---|---|
| Hunt & Reffert 2023 (VizieR `J/A+A/673/A114/clusters`) | Cluster ages, distances, membership counts | `astroquery.vizier` |
| Unified Cluster Catalogue (UCC, Zenodo record 14047628) | Cluster list with quality classes (`C3`), `N_50` members | CSV download via `pandas` |
| Lund catalogue (NASA HEASARC `lyngaclust.fits`) | Alternative age catalogue | FITS download |
| Gaia DR3 (VizieR `I/355/gaiadr3`) | Cluster-field photometry and parallaxes | `astroquery.vizier` |

## Notebook Walkthrough

| Cell | What it does | Data used |
|---|---|---|
| 0 | Installs `numpy`, `matplotlib`, `scipy`, `astroquery`, `astropy`, `pandas` | — |
| 1 | Downloads the UCC catalogue (16,179 clusters), keeps quality class `C3 == 1`, and selects the 100 clusters with the most members (`N_50`). Tries the Lund catalogue too. | Real |
| 2 | Fetches the Hunt & Reffert 2023 catalogue, converts `logAge` to Gyr, filters to valid ages, groups clusters by age band, plots age histograms, and exports a CSV. | Real |
| 3 | Queries Gaia for Pleiades/Hyades/M67, builds colour–magnitude diagrams, estimates ages from the turn-off and compares them with the literature. | **Simulated** (query failed) |
| 4 | Re-installs dependencies. | — |
| 5 | Gaia DR3 query with quality cuts (RUWE < 1.4, parallax error < 20%, parallax-based membership). Produces comparative and individual H-R diagrams plus an MSTO trend plot. | Real Gaia query, very few stars |
| 6 | Builds an age table with error bars and an uncertainty budget; defines Monte Carlo, bootstrap, χ² and parallax-propagation uncertainty routines. | **Hard-coded** table values |
| 7 | Attempts to run an external `generate_hr_diagrams.py`. | Script missing |
| 8 | Cleaner re-run of cell 5 for dissertation figures. | Real Gaia query, very few stars |
| 9–11, 14 | Stand-alone H-R diagram generators using synthetic stars, with isochrone-style curves and turn-off markers. | **Simulated** |
| 12 | Matplotlib sanity check (sine wave). | — |
| 13 | Quick test plot of random numbers. | Random |

## Outputs

Files written by the notebook (in the Colab working directory):

- `open_clusters_100_for_analysis.csv`: cluster sample for analysis
- `hr_diagrams_comparison.png/.pdf`, `hr_diagrams_comparative.png/.pdf`: 3-panel H-R diagrams
- `hr_diagram_pleiades|hyades|m67.png/.pdf` and `HR_Diagram_*.png/.pdf`: individual diagrams
- `msto_evolution_trend.png/.pdf`, `msto_evolution_summary.png/.pdf`: turn-off vs age summary
- `cluster_ages_with_errors.csv`: age table with error bars
- `uncertainty_budget.csv`: uncertainty budget table
- `hr_diagrams.png/.pdf`, `hr_diagram_test.png`: simple/test versions

## Data Provenance & Limitations

Be careful about what each result actually represents:

| Result | What it really is |
|---|---|
| Catalogue statistics (cells 1, 2) | Real data, but the Hunt & Reffert query returned only **44 clusters** with valid ages (29 after the reliability cut), far fewer than the full catalogue. The exported CSV is named `..._100_...` but holds 29 rows. |
| Age estimates in cell 3 (Pleiades 0.010 Gyr, Hyades 0.214 Gyr, M67 1.03 Gyr) | Computed from **simulated** stars after the Gaia query failed. They are 66–92% below literature values and should not be reported. |
| Gaia H-R diagrams (cells 5, 8) | Real query, but only **8 (Pleiades), 16 (Hyades) and 15 (M67)** member stars survive the cuts. That is too few for a reliable turn-off or age. |
| Age table with error bars (cell 6) | The "this study" ages (0.125, 0.650, 4.0 Gyr) and their errors are **typed in by hand**, not derived from the data. The Pleiades and M67 values are identical to the literature ones. |
| Uncertainty budget (cell 6) | The per-source error contributions are **illustrative inputs**, not measured quantities. |
| H-R diagrams in cells 9–11, 14 | **Synthetic** stars drawn from random distributions around assumed turn-off values. The "isochrones" are simple straight-line formulas, not PARSEC models, even where labelled "PARSEC". |

## Getting Started

### Prerequisites

- Python 3.10+ (tested on Colab with Python 3.12)
- Internet access (the notebook queries VizieR, Zenodo, HEASARC and the Gaia archive)

### Install

```bash
pip install numpy pandas matplotlib scipy astropy astroquery
```

### Run

1. Open `Astronomy.ipynb` in Google Colab or Jupyter.
2. Run the cells in order.
3. Generated figures and CSVs appear in the working directory.

Notes:

- Cell 11 sets `matplotlib.use('TkAgg')`, which fails on Colab and most servers. Remove it or use the default inline backend.
- Cell 7 calls `generate_hr_diagrams.py`, which is not part of the notebook. Skip it, or use cell 8, which contains the same logic.

## Known Issues

- **`Vizier.get_catalog` does not exist** in current `astroquery`, so cell 3 always falls back to simulated data. Use `Vizier.query_region(...)` as cells 5 and 8 do.
- **Lund catalogue FITS file fails to load** ("Empty or corrupt FITS file"); that branch silently returns an empty DataFrame.
- **Very small Gaia samples:** the parallax-only membership selection plus RUWE and parallax-error cuts leaves under 20 stars per cluster. Parallax alone cannot separate members from field stars; proper motions should be used too.
- **Fixed distances:** absolute magnitudes use hard-coded distances, with no extinction or reddening correction.
- **Turn-off age formula** in cell 6 is an ad-hoc empirical relation, not an isochrone fit.
- **Silent fallbacks:** `generate_simulated_data()` is called inside `except` blocks, and `warnings.filterwarnings('ignore')` hides the problem, so figures may be synthetic without any visible warning.
- **Duplicated code:** cells 5, 8, 9, 10, 11 and 14 repeat most of the same plotting logic.

## Suggested Next Steps

- Remove the simulated-data fallback, or make it raise a loud error so synthetic data can't end up in a figure.
- Select members using proper motion and parallax (or use published membership lists from Hunt & Reffert 2023 / Cantat-Gaudin) to get hundreds of stars per cluster.
- Apply reddening/extinction corrections.
- Fit real isochrones (PARSEC, MIST) to derive ages, instead of typing the values in.
- Compute the uncertainty budget from the actual fits and propagate it properly.
- Merge the repeated plotting cells into one function and add a `requirements.txt`.

## Acknowledgements

- [Gaia](https://www.cosmos.esa.int/gaia) mission data (ESA), accessed through [VizieR](https://vizier.cds.unistra.fr/)
- Hunt & Reffert (2023) open-cluster catalogue; Unified Cluster Catalogue (Perren et al.)
- Built with [Astropy](https://www.astropy.org/), [Astroquery](https://astroquery.readthedocs.io/), [Matplotlib](https://matplotlib.org/), [SciPy](https://scipy.org/) and [pandas](https://pandas.pydata.org/)

## License

Add a license of your choice (e.g., MIT) before publishing.
