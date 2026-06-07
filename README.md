# Spatial Regression Analysis of Technology Adoption in Nairobi SMEs

[![R](https://img.shields.io/badge/R-4.x-276DC3?style=flat&logo=r&logoColor=white)](https://www.r-project.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Status](https://img.shields.io/badge/Status-Complete-brightgreen)](https://github.com)
[![Spatial](https://img.shields.io/badge/Analysis-Spatial%20Econometrics-blue)](https://github.com)

A spatial econometric analysis examining the determinants of **technology adoption among Small and Medium Enterprises (SMEs) across Nairobi's sub-counties**. This project applies Spatial Lag (SAR), Spatial Error (SEM), and Spatial Durbin (SDM) models to account for geographic spillover effects in technology diffusion.

---

## Table of Contents

- [Project Overview](#-project-overview)
- [Research Context](#-research-context)
- [Repository Structure](#-repository-structure)
- [Dataset Description](#-dataset-description)
  - [Dependent Variables](#dependent-variables)
  - [Independent Variables](#independent-variables)
- [Methodology](#-methodology)
  - [Spatial Weight Matrices](#spatial-weight-matrices)
  - [Spatial Autocorrelation Testing](#spatial-autocorrelation-testing)
  - [Spatial Regression Models](#spatial-regression-models)
  - [Model Selection](#model-selection)
- [Prerequisites & Installation](#-prerequisites--installation)
- [How to Run](#-how-to-run)
- [Results Summary](#-results-summary)
- [Shapefile Data](#-shapefile-data)
- [Limitations](#-limitations)
- [Citation](#-citation)

---

## Project Overview

This study investigates how **ICT competency, digital skills, and financial access** influence technology adoption among Nairobi SMEs using spatial regression techniques. Standard regression models assume spatial independence — an unrealistic assumption in an urban context where neighbouring businesses influence one another. This analysis addresses that by:

1. Testing for **spatial autocorrelation** using Moran's I
2. Building **three competing spatial models** (SAR, SEM, SDM)
3. Selecting the best-fitting model via **Lagrange Multiplier tests and AIC comparison**
4. Visualising results on **GADM-sourced Nairobi sub-county shapefiles**

---

## Research Context

| Attribute | Details |
|-----------|---------|
| **Study Area** | Nairobi County, Kenya |
| **Unit of Analysis** | Sub-county level (administrative level 2) |
| **Focal Topic** | Technology adoption in the informal/SME sector |
| **Spatial Framework** | Queen contiguity, Rook contiguity, Distance-based (2 km) |
| **Primary Outcome** | Adoption of New Technology (`y`) and sub-dimensions (Access, Openness, Coping) |

---

## Repository Structure

```
spartial_regression/
│
├── Spatial_V3.1.Rmd          # Main R Markdown analysis notebook
├── Spatial_V3.1.html          # Rendered HTML report (knitted output)
├── Combined_Data.csv          # Survey dataset with geo-coordinates (n ≈ 1,215)
│
└── gadm41_KEN_shp/            # Kenya administrative boundary shapefiles (GADM v4.1)
    ├── gadm41_KEN_0.*         # National boundary (Level 0)
    ├── gadm41_KEN_1.*         # County boundaries (Level 1)
    ├── gadm41_KEN_2.*         # Sub-county boundaries (Level 2) ← used in analysis
    └── gadm41_KEN_3.*         # Ward boundaries (Level 3)
```

> **Note:** Shapefile formats include `.shp` (geometry), `.dbf` (attributes), `.shx` (index), `.prj` (projection), and `.cpg` (encoding). All levels use **WGS 84 (EPSG:4326)** coordinate reference system.

---

## Dataset Description

**File:** `Combined_Data.csv`  
**Observations:** ~1,215 SME survey responses  
**Coverage:** Nairobi sub-counties (Dagoreti, Embakasi, Kasarani, Langata, Westlands, etc.)  
**Geo-fields:** `lat` (latitude), `long` (longitude), `ADM2_EN` (sub-county name)

### Dependent Variables

All outcome variables were measured on an ordinal Likert scale (`Very Low` → `Very High`) and converted to numeric scores (1–5) for modelling.

| Variable | Label | Description |
|----------|-------|-------------|
| `y` | **Adoption** | Overall adoption of new technology |
| `y_11` | **Access** | Access to new technology |
| `y_12` | **Openness** | Openness, reception, and utilisation of new technology |
| `y_13` | **Coping** | Coping with modern technology ← *primary outcome in models* |

### Independent Variables

| Variable | Label | Type | Description |
|----------|-------|------|-------------|
| `x_1` | ICTComp | Continuous | Competency in ICT |
| `x_2` | Trainedu | Continuous | Willingness to pursue continuous training/education |
| `x_3` | Mdigtech | Continuous | Mastering digital transformation |
| `x_4` | Newtech | Binary (0/1) | Whether employees use new technology (Yes/No) |
| `x_5` | Techemployees | Continuous | Percentage of employees with digital technical skills |
| `x_6` | Moperate | Ordinal (1/2/3) | SME operating mode — Physical (1), Online (2), Hybrid (3) |
| `x_7` | — | Continuous | Additional continuous predictor |
| `x_8` | Fininstmain | Binary | Financial institutions as main financiers (0/1) |
| `x_9` | Finbarrier | Binary | Finance identified as a main business barrier (0/1) |

---

## Methodology

### Spatial Weight Matrices

Three spatial weight structures are constructed and visualised:

| Matrix Type | Method | Description |
|-------------|--------|-------------|
| **Queen Contiguity** | `poly2nb()` | Neighbours share any boundary point or vertex |
| **Rook Contiguity** | `poly2nb(queen=FALSE)` | Neighbours share a boundary edge only |
| **Distance-based** | `dnearneigh()` | Sub-counties within **2 km** of each other |

Row-standardised weights (`style = "W"`) are used throughout the regression stage so that the spatial lag is a weighted average of neighbours' values.

### Spatial Autocorrelation Testing

**Moran's I test** is applied to all four outcome variables under the Queen contiguity weights matrix:

```
H₀: No spatial autocorrelation (random spatial distribution)
H₁: Positive/negative spatial autocorrelation present
```

A statistically significant Moran's I motivates the use of spatial regression over OLS.

### Spatial Regression Models

All three models regress `y_13` (Coping) on predictors `x_1, x_4, x_5, x_7, x_8, x_9`:

---

#### 1. Spatial Lag Model (SAR / SLM)

Captures **spatial dependence in the outcome** — technology coping in one sub-county is directly influenced by coping levels in neighbouring sub-counties.

$$y = \rho W y + X\beta + \varepsilon$$

- **ρ (rho):** Spatial autoregressive coefficient
- Fitted with `lagsarlm()` from the `spatialreg` package

---

#### 2. Spatial Error Model (SEM)

Captures **spatial dependence in the error term** — unmeasured factors (e.g., local infrastructure, culture) that affect multiple adjacent sub-counties similarly.

$$y = X\beta + u, \quad u = \lambda W u + \varepsilon$$

- **λ (lambda):** Spatial error autocorrelation coefficient
- Fitted with `errorsarlm()` from the `spatialreg` package

---

#### 3. Spatial Durbin Model (SDM)

A richer model capturing **both direct and spillover effects** of predictors from neighbouring sub-counties.

$$y = \rho W y + X\beta + WX\theta + \varepsilon$$

- Includes spatially lagged independent variables alongside the spatial lag of `y`
- Fitted with `lagsarlm(..., type = "mixed")`

---

### Model Selection

Two complementary selection approaches are used:

**Lagrange Multiplier (LM) Tests** (`lm.LMtests`, `test = "all"`):
- `LMlag` — tests whether SAR structure is needed
- `LMerr` — tests whether SEM structure is needed
- Robust variants (`RLMlag`, `RLMerr`) account for the presence of both effects simultaneously

**AIC Comparison:**
```r
aic_values <- c(SAR = AIC(sar_model), SAE = AIC(sae_model), SDM = AIC(sdm_model))
sort(aic_values)
```
The model with the **lowest AIC** is preferred as the best-fitting, most parsimonious specification.

**Pre-modelling Diagnostics:**
- **VIF** (Variance Inflation Factor) via `car::vif()` to detect multicollinearity before fitting spatial models.

---

## Prerequisites & Installation

### R Version
Requires **R ≥ 4.0**. Install from [https://www.r-project.org/](https://www.r-project.org/).

### Required Packages

Install all dependencies in one block:

```r
install.packages(c(
  "sf",           # Simple Features: reading/writing spatial data
  "ggplot2",      # Data visualisation
  "dplyr",        # Data manipulation
  "spdep",        # Spatial dependence: weights matrices, Moran's I, LM tests
  "spatialreg",   # Spatial regression models: SAR, SEM, SDM
  "car"           # Companion to Applied Regression: VIF diagnostics
))
```

### Optional (for rendering the report)

```r
install.packages(c("rmarkdown", "knitr"))
```

---

## How to Run

### Option 1: Knit the R Markdown Report (Recommended)

1. Clone this repository:
   ```bash
   git clone https://github.com/<your-username>/spartial_regression.git
   cd spartial_regression
   ```

2. Open `Spatial_V3.1.Rmd` in **RStudio**.

3. Update the two file paths at the top of the notebook to match your local machine:
   ```r
   # Line ~20 — dataset
   nrb <- read.csv("Combined_Data.csv")

   # Line ~40 — shapefile
   kenya_lvl3 <- st_read("gadm41_KEN_shp/gadm41_KEN_2.shp")
   ```

4. Click **Knit → Knit to HTML** (or run `rmarkdown::render("Spatial_V3.1.Rmd")`).

### Option 2: Run Chunks Interactively

Open `Spatial_V3.1.Rmd` in RStudio and execute code chunks sequentially using **Ctrl+Shift+Enter**. This is recommended when exploring intermediate outputs (maps, Moran plots, model summaries).

### Option 3: View Pre-rendered Output

Open `Spatial_V3.1.html` directly in any web browser — no R installation required.

---

## Results Summary

The analysis follows a structured pipeline:

```
Survey Data (CSV) + Nairobi Shapefile
        ↓
Ordinal Encoding → sf Conversion → Spatial Join
        ↓
Embakasi Sub-county Merge → NA Removal
        ↓
Queen / Rook / Distance Weight Matrices
        ↓
Moran's I Tests on y, y_11, y_12, y_13
        ↓
VIF Check → SAR / SEM / SDM Model Fitting
        ↓
LM Tests + AIC Comparison → Best Model Selection
```

Key modelling decisions:
- The **primary outcome is `y_13` (Coping with modern technology)**, chosen as the final dependent variable for the comparative model stage.
- The multiple Embakasi sub-divisions in GADM Level 2 are **dissolved into a single "Embakasi" polygon** to match the survey's administrative coding.
- Rows with missing values are **dropped via `na.omit()`** after the spatial join.

---

## Shapefile Data

The shapefiles are sourced from the **Global Administrative Areas (GADM) v4.1** database:

| Level | File | Coverage |
|-------|------|----------|
| 0 | `gadm41_KEN_0.shp` | National boundary |
| 1 | `gadm41_KEN_1.shp` | 47 Counties |
| **2** | **`gadm41_KEN_2.shp`** | **Sub-counties ← used in analysis** |
| 3 | `gadm41_KEN_3.shp` | Wards |

- **CRS:** WGS 84, EPSG:4326
- **Source:** [https://gadm.org/download_country.html](https://gadm.org/download_country.html)
- **License:** Free for academic and non-commercial use. Redistribution requires attribution to GADM.

> The full Kenya Level 3 shapefile (`gadm41_KEN_3.shp`) is ~25 MB and is included in the repository for completeness, but **only Level 2** is used in the analysis.

---

## Limitations

- **Ecological fallacy:** Variables are aggregated to sub-county level; individual-level inference should be made with caution.
- **Small n:** After the spatial join and NA removal, the number of spatial units equals the number of Nairobi sub-counties (~10–15), which limits statistical power.
- **Ordinal-to-numeric conversion:** Likert-scale variables are treated as interval-scale integers, which assumes equidistant categories.
- **Path hardcoding:** The original `.Rmd` file contains absolute local paths that must be updated before running on a new machine (see [How to Run](#-how-to-run)).
- **Single time point:** The dataset is cross-sectional; causal inference is not established.

---

## References

- Anselin, L. (1988). *Spatial Econometrics: Methods and Models*. Kluwer Academic Publishers.
- Bivand, R., & Piras, G. (2015). Comparing implementations of estimation methods for spatial econometrics. *Journal of Statistical Software*, 63(18), 1–36.
- GADM (2022). *Global Administrative Areas, Version 4.1*. University of California, Davis. Retrieved from https://gadm.org
- LeSage, J. P., & Pace, R. K. (2009). *Introduction to Spatial Econometrics*. CRC Press.

---

## Citation

If you use this analysis or adapt the code, please cite:

```bibtex
@misc{austine2025spatial,
  author       = {Austine, Stephen},
  title        = {Spatial Regression Analysis of Technology Adoption in Nairobi SMEs},
  year         = {2025},
  howpublished = {\url{https://github.com/<your-username>/spartial_regression}},
  note         = {Course project, STA3010 Statistical Modeling}
}
```

---

## Contact

**Stephen Austine**  
Data Scientist · ML Developer  
GitHub: [@Stephen-Austine](https://github.com/Stephen-Austine)

---
