# Design Dashboard

This repository contains the **Design Dashboard**, a climate forecasting and risk assessment tool designed to support humanitarian aid decisions and agricultural planning.

## Overview

The Design Dashboard provides:
- **Climate forecasts** with probability of exceedance analysis
- **Risk assessment** for humanitarian aid decision-making
- **Historical validation** of forecast performance
- **Multi-country support** for regions in Africa and beyond
- **Interactive data visualization** with maps, charts, and tables
- **Local shapefile support** for geographic boundaries without database connectivity

## Quick Start Guide

This guide will walk you through setting up the Design Dashboard from scratch, including data preparation, environment setup, and configuration.

### Prerequisites

- **Python 3.9+** and **Conda** installed
- **Git** for cloning the repository
- **Basic knowledge** of Python and command line operations

### Step 1: Clone and Setup Repository

```bash
# Clone the repository
git clone https://github.com/nitinmagima/python-maproom-djibouti.git
cd python-maproom-djibouti

# Navigate to the fbfmaproom directory
cd fbfmaproom
```

### Step 2: Install Conda Environment

**Choose the appropriate lock file for your operating system:**

- **Linux (64-bit):** `conda-linux-64-design-dashboard.lock`
- **macOS (Intel/64-bit):** `conda-osx-64-design-dashboard.lock`
- **macOS (Apple Silicon/ARM64):** `conda-osx-arm64-design-dashboard.lock`
- **Windows (64-bit):** `conda-win-64-design-dashboard.lock`

```bash
# Create and activate the conda environment using the appropriate lock file

# For Linux:
conda create -n designdashboard --file conda-linux-64-design-dashboard.lock

# For macOS Intel:
conda create -n designdashboard --file conda-osx-64-design-dashboard.lock

# For macOS Apple Silicon:
conda create -n designdashboard --file conda-osx-arm64-design-dashboard.lock

# For Windows:
conda create -n designdashboard --file conda-win-64-design-dashboard.lock

# Activate the environment
conda activate designdashboard

### Step 3: Data Preparation

The application requires three types of data files:

**Note:** If you want to learn how to generate the forecast data yourself, check the [PyCPT 2.5 Seasonal Forecast User Guide](https://iri-pycpt.github.io/PyCPT2-Seasonal-Forecast-User-Guide/intro.html). All the forecasts for Djibouti can be found in the `python_maproom_djibouti` submodule folder.

#### 3.1 Forecast Data (Zarr Format)

**Source:** NetCDF forecast files in `data/original-data/djibouti/prcp-jas-v4/`

**Process:** Convert NetCDF files to Zarr format for efficient access

```bash
# Navigate to data conversion scripts
cd data-conversion-scripts

# Convert forecast data to Zarr format
# Format: python zarrify-forecast.py [country]/[dataset-name]
python zarrify-forecast.py djibouti/prcp-jas-v4

# This script will:
# - Read NetCDF files from data/original-data/djibouti/prcp-jas-v4/
# - Convert them to Zarr format
# - Save to data/djibouti/prcp-jas-v4.zarr/
```

**Expected Output:** `data/djibouti/prcp-jas-v4.zarr/` directory with Zarr-formatted forecast data

#### 3.2 Bad Years Data (Zarr Format)

**Source:** Bad years classification data (CSV files)

**Process:** Convert CSV files to Zarr format for consistency

```bash
# Convert bad years data to Zarr format
# Format: python zarrify-bad-years.py [country]/[filename] [target_month_center]
python zarrify-bad-years.py djibouti/bad-years-v2-jas 7.5
```

**Important Notes:**
- **Don't add `.csv`** to the filename when running the script
- **Target month center** (7.5) represents the center of the target season (found in YAML configuration)
- **File location:** The script expects CSV files in the appropriate country directory

**Expected Output:** `data/djibouti/bad-years-v2-jas.zarr/` directory with Zarr-formatted bad years data

**Dataset Configuration Guidelines:**

The application supports two types of bad years data with different interpretations:

1. **Categorical Classification (format: bad):**
   ```yaml
   bad-years:
     label: Bad Years Classification
     format: bad  # Categorical yes/no classification
     # No units declaration needed
   ```

2. **Numerical Ranking (format: number0 + units: rank):**
   ```yaml
   bad-years-rank-v2:
     label: Bad Years Ranking
     format: number0  # Integer with zero decimals
     units: rank      # Ordinal ranking (1st, 2nd, 3rd, etc.)
     lower_is_worse: yes  # Higher number = lower rank (worse)
   ```

**Key Differences:**
- **format: bad** = Categorical classification (Bad Year vs Not Bad Year)
- **format: number0 + units: rank** = Numerical ranking (1 = worst, 2 = second worst, etc.)
- **lower_is_worse: yes** = Used with ranks where higher numbers indicate worse conditions

#### 3.3 Geographic Boundaries (Shapefiles)

**Source:** GADM shapefiles for Djibouti administrative boundaries

**Location:** `data/djibouti/dji_adm_gadm_2022_shp/`

**Files Included:**
- `dji_admbnda_gadm_adm0_2022.shp` - National boundaries
- `dji_admbnda_gadm_adm1_2022.shp` - Regional boundaries  
- `dji_admbnda_gadm_adm2_2022.shp` - District boundaries

**Note:** These shapefiles are already included in the repository and don't require conversion.

### Step 4: Verify Data Structure

Ensure your data directory structure looks like this:

```
fbfmaproom/
├── data/
│   └── djibouti/
│       ├── prcp-jas-v4.zarr/          # Forecast data
│       ├── bad-years-v2-jas.zarr/     # Bad years data
│       └── dji_adm_gadm_2022_shp/     # Shapefiles
│           ├── dji_admbnda_gadm_adm0_2022.shp
│           ├── dji_admbnda_gadm_adm1_2022.shp
│           └── dji_admbnda_gadm_adm2_2022.shp
```



### Step 5: Configuration

#### 5.1 Update Configuration File

Edit `fbfmaproom-sample.yaml` to match your data paths:

```yaml
# Key configuration sections to verify:

data_root: ./data  # Ensure this points to your data directory

shapes:
  - name: National
    local_file: dji_admbnda_gadm_adm0_2022.shp
    key_field: ADM0_PCODE
    label_field: ADM0_EN
    # ... database fallback SQL ...

  - name: Regional  
    local_file: dji_admbnda_gadm_adm1_2022.shp
    key_field: ADM1_PCODE
    label_field: ADM1_EN
    # ... database fallback SQL ...

  - name: District
    local_file: dji_admbnda_gadm_adm2_2022.shp
    key_field: ADM2_PCODE
    label_field: ADM2_EN
    # ... database fallback SQL ...

datasets:
  forecasts:
    pnep-v4:
      path: djibouti/prcp-jas-v4.zarr  # Verify this path
      # ... other configuration ...

  observations:
    bad-years-rank-v2:
      path: djibouti/bad-years-v2-jas.zarr  # Verify this path
      # ... other configuration ...
```



### Step 6: Run the Application

#### 6.1 Start the Application

```bash
# Run the application with configuration and Python path
PYTHONPATH=$PYTHONPATH:$(pwd) CONFIG=fbfmaproom-sample.yaml python fbfmaproom.py
```

#### 6.2 Access the Application

The application will start on a default port (usually 8050). Check the terminal output for the exact URL.

Open your web browser and navigate to:
- **Main Application:** `http://localhost:8050/fbfmaproom/djibouti` (or the port shown in terminal)
- **Regions API:** `http://localhost:8050/fbfmaproom/regions?country=djibouti&level=0` (or the port shown in terminal)

**Note:** If port 8050 is already in use, the application will automatically use the next available port. Check the terminal output for the actual URL.

### Step 7: Verify Installation

- Navigate to `http://localhost:8050/fbfmaproom/djibouti`
- Verify the map loads without errors
- Test clicking on different regions
- Verify forecast data displays correctly

### Support

Contact Nitin Magima (nm2996@columbia.edu) or the Financial Instruments Sector Team, NCDP (fist@iri.columbia.edu) for support.



