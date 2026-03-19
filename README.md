# MSF Tool (Python)
**Developer:** [Yan Zhong](https://sites.google.com/view/yanzhong-geo), [University of Geneva](https://c-cia.ch/)  
**E-mails:** yan.zhong@unige.ch | yan.zhong.geo@gmail.com  
**Toolbox file:** `MSF Python.atbx`  
**Last updated:** 08.03.2026  
**Language:** English  
**Coding language:** Python  
**Operating System:** Windows (ArcGIS Pro)  
**Installation Time:** ~1 second  

---

## Overview

This toolbox contains three Python script tools for **mass-flow runout modelling** and **watershed delineation** in ArcGIS Pro. The MSF tools implement the **Modified Single Flow direction (MSF) model** developed by [Christian Huggel](https://nhess.copernicus.org/articles/3/647/2003/nhess-3-647-2003.html) et al. in 2003, simulating gravity-driven hazards such as **glacial lake outburst floods (GLOFs)**, **debris flows**, and **landslides** using flow-direction-weighted path distance analysis.

| Script | Description |
|--------|-------------|
| **MSF Python** | Per-AOI runout modelling from polygon inputs |
| **MSF Python Basin** | Multi-AOI basin-wide runout with aggregated output |
| **Watershed Delineation** | Catchment delineation from pour points |

---

## Tool 1: MSF Python

### Inputs

| Parameter | Type | Description |
|-----------|------|-------------|
| DEM | Raster Dataset | Digital Elevation Model. |
| AOI | Feature Layer | AOI polygons. |
| Output Folder | Folder | Directory to save output rasters and temporary files. |
| Tanα | String (optional) | Minimum runout slope threshold (default = `0.05`). Pixels with `pgse < Tanα` are masked out. |

> **Note:** The model is tolerant of input data formats: DEMs can be provided either projected or unprojected, and AOI polygon sets can be used directly without specifying unique IDs.  
> **Note:** Demo data are provided in the tools package for model demonstration purposes.  
> **Expected run time:** ~3 minutes on a standard desktop computer.

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `path_<ID>.tif` | Raster | Runout slope raster for each AOI, filtered by Tanα. One file per AOI. |
| `MSF_temp/` | Folder | Intermediate files (dem_int, dem_cr, fldir, fldir_deg, costalloc_dem, h_diffse, etc.). Can be deleted after a successful run. |

### Workflow

1. Prepare a DEM and an AOI polygon layer covering the study area.
2. Set the output folder and optionally adjust the **Tanα** threshold.
3. Run the tool:
   - **PART 0** — Projection check: if DEM or AOI are in geographic CRS or non-meter units, they are automatically reprojected to the appropriate UTM zone.
   - **PART 1** — DEM preprocessing: `Int → Fill → Int → FlowDirection → Log2 → fldir_deg`.
   - **PART 2** — Per-AOI iteration: `Feature to Raster → Cost Allocation → Path Distance → Extract by Mask → z_diffse → pgse → filter by Tanα → save`.
4. Collect output rasters (`path_<ID>.tif`) from the output folder.

---

## Tool 2: MSF Python Basin

An extended version of MSF Python designed for **basin-wide analysis** with multiple AOIs. Rather than producing one raster per AOI, it aggregates all runout paths into a single accumulated output. AOIs are automatically grouped to avoid redundant computation where runout zones would overlap.

### Inputs

| Parameter | Type | Description |
|-----------|------|-------------|
| DEM | Raster Dataset | Digital Elevation Model. |
| AOI | Raster Dataset | AOI raster (positive values mark source areas). |
| Output Folder | Folder | Directory to save output rasters and temporary files. |
| Tanα | String (optional) | Minimum runout slope threshold (default = `0.05`). |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `paths_all.tif` | Raster | Accumulated runout slope raster across all AOIs. |
| `paths_all_normalized.tif` | Raster | `paths_all.tif` normalized to [0, 1] by its maximum value. |
| `MSF_temp/` | Folder | Intermediate files. Can be deleted after a successful run. |

### Workflow

1. **PART 0** — Projection check: same auto-reprojection logic as MSF Python.
2. **PART 1** — DEM preprocessing: identical to MSF Python.
3. **PART 2** — AOI grouping: individual AOI patches are identified and assigned to non-overlapping groups using a 50-pixel dilation buffer, minimising redundant Cost Allocation / Path Distance runs.
4. **PART 3** — MSF per group: runout is computed per group and accumulated into a single array (`ei_arr`).
5. **PART 4** — Outputs saved as `paths_all.tif` and `paths_all_normalized.tif`.

---

## Tool 3: Watershed Delineation

Delineates catchment boundaries from a DEM and a set of pour points, outputting watershed polygons with area calculations.

### Inputs

| Parameter | Type | Description |
|-----------|------|-------------|
| DEM | Raster Dataset | Digital Elevation Model. |
| Pour Points | Feature Layer | Point layer marking watershed outlets. |
| Output Folder | Folder | Directory to save outputs and temporary files. |
| Snap Distance | String (optional) | Distance (m) to snap pour points to the nearest flow accumulation maximum (default = `500`). |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `watershed.shp` | Shapefile | Watershed polygons with an `Area_km2` field. |
| `WS_temp/` | Folder | Intermediate files (dem_fill, fldir, flacc, snapped, watershed raster). Can be deleted after a successful run. |

### Workflow

1. **PART 0** — Projection check: DEM and pour points are reprojected to UTM if needed.
2. **PART 1** — DEM preprocessing: `Fill → FlowDirection → FlowAccumulation`.
3. **PART 2** — Pour points are snapped to the nearest high-accumulation cell within the snap distance.
4. **PART 3** — Watershed delineation using the snapped pour points and flow direction raster.
5. **PART 4** — Watershed raster converted to polygon; `Area_km2` field calculated and reported.

---

## General Notes

**Requires the Spatial Analyst extension** in ArcGIS Pro for all three tools.  
All tools share the same automatic UTM reprojection logic — inputs in geographic CRS or non-meter projected CRS are handled transparently.

---

## License

See the [LICENSE](LICENSE) file for full terms.
