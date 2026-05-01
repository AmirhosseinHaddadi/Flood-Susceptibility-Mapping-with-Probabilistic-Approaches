# 🌊 Flood Susceptibility Mapping using FR & WoE

[![Python 3.8+](https://img.shields.io/badge/Python-3.8+-blue.svg)](https://www.python.org/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Rasterio](https://img.shields.io/badge/Rasterio-1.3+-orange.svg)](https://rasterio.readthedocs.io/)

A robust geospatial pipeline for flood susceptibility mapping using **Frequency Ratio (FR)** and **Weights of Evidence (WoE)** methods. Memory‑safe windowed processing for large AOIs.

---

## 📋 Table of Contents
- [Overview](#overview)
- [Features](#features)
- [Methodology](#methodology)
- [Input Data](#input-data)
- [Outputs](#outputs)
- [Configuration](#configuration)
- [Usage](#usage)
- [Requirements](#requirements)
- [Citation](#citation)

---

## 🔍 Overview

This tool processes multi‑layer geospatial data (DEM, slope, TWI, NDVI, LULC, rainfall, etc.) to produce high‑resolution flood susceptibility maps using two statistical bivariate methods:

- **Frequency Ratio (FR)** – based on the ratio of flood occurrence in each class
- **Weights of Evidence (WoE)** – log‑linear Bayesian framework

All operations are **windowed/tiled** to handle huge rasters without memory issues.

---

## ✨ Features

| Category | Capabilities |
|----------|---------------|
| **Sampling** | GRTS‑like spatial sampling, stratified (flood/non‑flood), configurable size |
| **Binning** | Automatic quantile‑based bins for continuous layers, categorical preserved |
| **Statistics** | FR weights, WoE weights (W⁺, W⁻, Contrast) |
| **Validation** | Train/validation split, ROC curves, AUC (Success & Prediction rates) |
| **Multicollinearity** | Correlation matrix + heatmap, VIF (Variance Inflation Factor) |
| **Mapping** | Raw score, normalized [0‑1], quantile‑classified (5 classes) rasters |
| **Memory** | Block‑wise I/O, configurable tile size |

---

## 🧠 Methodology
┌─────────────────┐ ┌──────────────────┐ ┌─────────────────┐
│ Input Layers │────▶│ Spatial Sampling │────▶│ Train/Val Split │
│ (raster files) │ │ (flood/non‑flood) │ │ (75% / 25%) │
└─────────────────┘ └──────────────────┘ └─────────────────┘
│
▼
┌─────────────────┐ ┌──────────────────┐ ┌─────────────────┐
│ Susceptibility │◀────│ FR / WoE Tables │◀────│ Global Stats │
│ Maps (TIFF) │ │ (weights per bin)│ │ (all AOI) │
└─────────────────┘ └──────────────────┘ └─────────────────┘
│
▼
┌─────────────────────────────────────────────────────────────────┐
│ Output: Raw → Normalized → 5‑Class (Quantile) → AUC/ROC/Plots │
└─────────────────────────────────────────────────────────────────┘

text

---

## 📥 Input Data

| Layer Type | Examples | Handling |
|------------|----------|----------|
| **Continuous** | DEM, Slope, TWI, TRI, SPI, STI, Distance to river, NDVI, Rainfall | Quantile bins (default 10) |
| **Categorical** | LULC, Soil/Lithology | Preserved original classes |

**Required base files:**
- `flooded_mask_otsu_fulls.tif` – binary flood inventory (1 = flood, 0 = non‑flood)
- `Subbasins_HydroBASINS_L8.shp` – AOI boundary shapefile
- All predictive layers (see `CONFIG["LAYERS"]`)

All layers are automatically reprojected to a **UTM grid at 10m resolution**.

---

## 📤 Outputs

### Rasters
| File | Description |
|------|-------------|
| `flood_mask_10m_uint8.tif` | Flood mask resampled to 10m |
| `susceptibility_FR_raw.tif` | FR raw sum of ln(FR) |
| `susceptibility_FR_norm.tif` | FR normalized [0‑1] |
| `susceptibility_FR_class5.tif` | FR 5 quantile classes |
| `susceptibility_WoE_raw.tif` | WoE raw sum of W⁺ |
| `susceptibility_WoE_norm.tif` | WoE normalized [0‑1] |
| `susceptibility_WoE_class5.tif` | WoE 5 quantile classes |

### Tables (Excel + CSV)
| File | Content |
|------|---------|
| `FR_weights.xlsx` | FR per bin: N_flood, N_total, P_flood, P_area, FR, lnFR |
| `WoE_weights.xlsx` | WoE per bin: W⁺, W⁻, Contrast |
| `samples_2000_FR_WoE.xlsx` | Sample points with all layer values |
| `AUC_summary.xlsx` | Train/validation AUC for FR & WoE |
| `Success_Prediction_Rates.xlsx` | Success (train) & Prediction (val) rates |
| `correlation_matrix.xlsx` | Pearson correlation between layers |
| `VIF_table.xlsx` | Variance Inflation Factor diagnostics |

### Plots
- ROC curves (`ROC_FR_Validation.png`, etc.)
- Correlation heatmap
- VIF bar chart
- Downsampled susceptibility maps (normalized & classified)

### Shapefile
- `sample_points_shp/samples_2000_FR_WoE.shp` – spatial samples

---

## ⚙️ Configuration

Edit the `CONFIG` dictionary in the script:

```python
CONFIG = {
    # Paths
    "FLOOD_MASK_TIF": r"path/to/flooded_mask.tif",
    "AOI_SHP": r"path/to/aoi.shp",
    "LAYERS_DIR": r"path/to/layers/",
    
    # Layer mapping (key: filename without extension)
    "LAYERS": {
        "DEM": "DEM_30m_AOI",
        "slope": "slope",
        "TWI": "twi_proj",
        # ... add all layers
    },
    
    # Sampling
    "N_SAMPLES_TOTAL": 2000,      # total sample points
    "POS_RATIO": 0.5,             # proportion flood points
    "VALIDATION_SIZE": 0.25,      # validation split
    
    # Binning
    "N_BINS_CONT": 10,            # bins for continuous layers
    "MIN_VALID_SAMPLES_BINNING": 50,
    
    # Processing
    "TARGET_RES_M": 10.0,         # output resolution
    "BLOCK_SIZE": 512,            # tile size (pixels)
    "SMOOTH": 0.5,                # Laplace smoothing
    
    # Categorical layers list
    "CATEGORICAL_LAYERS": ["LULC", "soil_lithology"],
}
🚀 Usage
1. Clone & Install
bash
git clone https://github.com/yourusername/flood-susceptibility-fr-woe.git
cd flood-susceptibility-fr-woe
pip install -r requirements.txt
2. Prepare your data
Place all rasters in LAYERS_DIR

Update CONFIG paths and layer names

Ensure all layers overlap with AOI

3. Run
bash
python fr_woe_mapping.py
4. Check outputs
All results saved in OUT_DIR with automatic timestamped log file.

📦 Requirements
text
rasterio>=1.3.0
geopandas>=0.12.0
numpy>=1.21.0
pandas>=1.3.0
scikit-learn>=1.0.0
statsmodels>=0.13.0
matplotlib>=3.5.0
pyproj>=3.3.0
shapely>=1.8.0
openpyxl>=3.0.0
📖 Citation
If you use this code, please cite:

text
[Your paper details here – EGU26 presentation]
📧 Contact
Amir – [Your email]
[Your affiliation]

🙏 Acknowledgments
HydroBASINS for watershed data

Copernicus / USGS for open geospatial data

⚠️ Notes
Memory usage is ~O(block_size²) – adjust BLOCK_SIZE for your RAM

All intermediate rasters use DEFLATE compression + tiling

For very large AOIs (>100k × 100k), increase CANDIDATE_OVERSAMPLE_FACTOR

Made with ❤️ for flood risk assessment
