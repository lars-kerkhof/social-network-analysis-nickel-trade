# SNA — Global Nickel Trade Network Analysis

Code and datasets for the course Social Network Analysis (SNA).

This project models the **global nickel trade** as a weighted, directed network and
uses social network analysis methods (MRQAP and ERGM) to test whether downstream
demand (electric vehicles and batteries), environmental performance, economic size,
logistics performance, and geographic distance explain trade ties between countries.

---

## Research question

> What country-level and dyadic factors shape the structure of the global nickel
> trade network, and how do downstream EV and battery export capabilities relate
> to a country's position in that network?

The network's nodes are countries (ISO3 codes) and edges are nickel trade flows
(`primaryValue` from UN Comtrade). Node attributes and dyadic covariates are
brought in from the datasets listed below.

---

## Repository structure

```
SNA/
├── README.md
├── SNA-nickeltrade_notebook.Rmd   # Main analysis notebook (R Markdown)
├── Report.pdf                     # Contains the assignment report
└── datasets/
    ├── TradeData_nickel.csv           # UN Comtrade nickel trade flows (edges)
    ├── TradeData_EV_exports.csv       # Electric vehicle exports per country
    ├── TradeData_Battery_exports.csv  # Battery exports per country
    ├── EPI_data.xlsx                  # Environmental Performance Index
    ├── GDP_data.xlsx                  # GDP per country (USD)
    ├── LSCI.csv                       # UNCTAD Liner Shipping Connectivity Index
    └── dist_cepii.dta                 # CEPII bilateral distance data
```

---

## Data sources

| File | Source | Used as |
|---|---|---|
| `TradeData_nickel.csv` | UN Comtrade | Edge list (directed, weighted by `primaryValue`) |
| `TradeData_EV_exports.csv` | UN Comtrade | Node attribute (downstream EV export capacity) |
| `TradeData_Battery_exports.csv` | UN Comtrade | Node attribute (downstream battery export capacity) |
| `EPI_data.xlsx` | Yale EPI | Node attribute (environmental performance score) |
| `GDP_data.xlsx` | World Bank | Node attribute (economic size) |
| `LSCI.csv` | UNCTAD | Node attribute (shipping connectivity) |
| `dist_cepii.dta` | CEPII GeoDist | Dyadic covariate (bilateral distance) |

---

## Methodology

The notebook (`SNA-nickeltrade_notebook.Rmd`) is organized as follows:

1. **Load & clean trade data** — read UN Comtrade nickel flows, drop NAs, drop
   self-loops, drop aggregate codes (`W00`, `_X`, `S19`), and aggregate duplicate
   edges by summing weights.
2. **Build the network** — construct a directed, weighted adjacency matrix and
   convert to an `igraph` object using ISO3 country codes as vertex names.
3. **Attach node attributes** — merge EPI, GDP, LSCI, EV exports and battery
   exports onto the vertices, with a synonyms table to harmonize country names
   to ISO3 codes (e.g. "Türkiye" → `TUR`, "Hong Kong SAR" → `HKG`).
4. **Build the distance matrix** — load CEPII bilateral distances and align them
   with the network's vertex order as a dyadic covariate.
5. **Impute missing attribute values** where needed.
6. **Study 1 — MRQAP** — multiple regression quadratic assignment procedure
   (`sna::netlm`) on the weighted trade matrix, with dyad-level goodness-of-fit
   checks.
7. **Study 2 — ERGM** — exponential random graph models (`ergm`) on a binarized
   network, fit in several specifications (Models 1, 2, 3, 4a, 4b, 4b_extra)
   with MCMC diagnostics, GOF plots, and a final comparison table.

---

## Requirements

- **R** (≥ 4.2 recommended)
- **RStudio** (for knitting the `.Rmd`)
- R packages:
  - `readr`, `readxl`, `haven` — data import
  - `igraph` — network construction and manipulation
  - `network`, `sna` — `netlm` / MRQAP
  - `ergm` — exponential random graph models

Install everything in one go:

```r
install.packages(c(
  "readr", "readxl", "haven",
  "igraph", "network", "sna", "ergm"
))
```

---

## How to run

1. Clone the repository:

   ```bash
   git clone https://github.com/lars-kerkhof/sna.git
   cd sna
   ```

2. Open `SNA-nickeltrade_notebook.Rmd` in RStudio.

3. **Update the file paths** at the top of the notebook. The first chunk
   currently hard-codes absolute Windows paths (e.g.
   `C:/Users/verab/OneDrive - TU Eindhoven/.../Data/TradeData_nickel.csv`).
   Repoint them to the local `datasets/` folder, for example:

   ```r
   csv_path  <- "datasets/TradeData_nickel.csv"
   epi_path  <- "datasets/EPI_data.xlsx"
   gdp_path  <- "datasets/GDP_data.xlsx"
   lsci_path <- "datasets/LSCI.csv"
   ev_path   <- "datasets/TradeData_EV_exports.csv"
   batt_path <- "datasets/TradeData_Battery_exports.csv"
   dist_path <- "datasets/dist_cepii.dta"
   ```

4. Knit the notebook (`Knit` button in RStudio), or run chunks interactively
   from top to bottom. The ERGM chunks can take several minutes depending on
   the specification.

---

## Authors

Group 3 — Social Network Analysis for Data Science, TU Eindhoven.
