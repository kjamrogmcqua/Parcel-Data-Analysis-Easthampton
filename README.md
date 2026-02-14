# Easthampton Parcel Data Analysis

An R workflow for reading MassGIS Level 3 Assessor parcel geodatabases (.gdb), joining parcel geometry to property assessment data, and producing choropleth maps. Built for Easthampton, MA but adaptable to any Massachusetts municipality.

## What It Does

1. **Reads** an ESRI file geodatabase and handles curved geometry types
2. **Joins** parcel boundaries to assessment attributes and use code descriptions
3. **Derives** analysis columns (land use category, lot size in acres, value per acre)
4. **Maps** assessed values, land use, and value density as choropleths
5. **Exports** the joined dataset to RDS, GeoPackage, CSV, and XLSX

## Data Source

Parcel data comes from the [MassGIS Level 3 Assessor Parcels](https://www.mass.gov/info-details/massgis-data-property-tax-parcels) program. Each municipality has a standardized geodatabase containing:

- **TaxPar** — parcel boundary polygons
- **Assess** — property assessment attributes (values, use codes, owner, address, year built)
- **UC_LUT** — use code lookup table (code to description)
- **LUT** — general lookup table (polygon type, legal type)

Download your municipality's GDB from the MassGIS link above and place it in `data/gdb/`.

## Project Structure

```
Parcel Data Analysis/
├── easthampton_parcels.Rmd      # Main notebook — read, join, map, export
├── data/
│   ├── gdb/                     # MassGIS geodatabase (raw, not tracked by git)
│   └── processed/               # Outputs: .rds, .gpkg, .csv, .xlsx
└── output/
    └── maps/                    # Choropleth PNGs
```

## Requirements

**R packages:**

```r
install.packages(c("tidyverse", "sf", "scales", "here", "writexl", "conflicted"))
```

The `sf` package requires GDAL, GEOS, and PROJ system libraries. On macOS: `brew install gdal`. On Ubuntu: `sudo apt install libgdal-dev`.

## Quick Start

1. Clone the repo and open `Parcel Data Analysis.Rproj` in RStudio
2. Download a MassGIS parcel GDB and place it in `data/gdb/`
3. Open `easthampton_parcels.Rmd`
4. Update `gdb_path` and the layer names if using a different municipality
5. Knit the notebook or run all chunks

## Adapting to Another Municipality

Each MassGIS parcel GDB follows the same schema with a municipality prefix (e.g., `M087` for Easthampton). To adapt:

1. Replace the GDB file in `data/gdb/`
2. Update the layer name prefix in `st_read()` calls (e.g., `M087TaxPar` to `M001TaxPar`)
3. Update map titles

The join keys (`MAP_PAR_ID` to `PROP_ID`) and use code classification logic are standardized across all Massachusetts municipalities.

## Example Output

The notebook produces four choropleth maps:

| Map | Description |
|-----|-------------|
| Total Assessed Value | Log-scale assessed value per parcel |
| Land Use Category | Broad classification (residential, commercial, etc.) |
| Residential Value per Acre | Fiscal density of residential parcels |
| Building Value | Building improvements value per parcel |

## Notes

- ESRI geodatabases may contain curved geometry types (MULTISURFACE, CURVEPOLYGON). The notebook linearizes these with `sf::st_cast()` before export or plotting.
- The CRS for Massachusetts data is EPSG:26986 (NAD83 / Massachusetts Mainland).
- Use code categories follow the Massachusetts standard: first digit maps to broad classification (1 = Residential, 3 = Commercial, 4 = Industrial, etc.).
