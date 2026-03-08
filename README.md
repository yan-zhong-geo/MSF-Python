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

This is a Python version of **Modified Single Flow direction (MSF) model** developed by [Christian Huggel](https://nhess.copernicus.org/articles/3/647/2003/nhess-3-647-2003.html) et al. in 2003.

This Python script simulates **mass-flow runout** from a digital elevation model (DEM) and a set of areas of interest (AOI) polygons using flow-direction-weighted path distance analysis. The approach can be applied to model a range of gravity-driven hazards, including **glacial lake outburst floods (GLOFs)**, **debris flows**, and **landslides**.

---
## Inputs

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| DEM | Raster Dataset | Yes | Digital Elevation Model. If geographic CRS (degrees) or non-meter units, automatically reprojected to UTM. |
| AOI | Feature Layer | Yes | AOI polygons. |
| Output Folder | Folder | Yes | Directory to save output rasters and temporary files. |
| Tanα | String | No | Minimum runout slope threshold (default = `0.05`). Pixels with `pgse < Tanα` are masked out. |

> **Note:** The model is tolerant of input data formats: DEMs can be provided either projected or unprojected, and AOI polygon sets can be used directly without specifying unique IDs.

---
## Outputs

| Output | Type | Description |
|--------|------|-------------|
| `path_<ID>.tif` | Raster | Runout slope raster for each lake, filtered by Tanα. One file per lake. |
| `MSF_temp/` | Folder | Intermediate files (dem_int, dem_cr, fldir, fldir_deg, costalloc_dem, h_diffse, etc.). Can be deleted after a successful run. |

---
## Workflow

1. Prepare a DEM and an AOI polygon layer covering the study area.
2. Set the output folder and optionally adjust the **Tanα** threshold.
3. Run the tool:
   - **PART 0** — Projection check: if DEM or AOI are in geographic CRS or non-meter units, they are automatically reprojected to the appropriate UTM zone.
   - **PART 1** — DEM preprocessing: `Int → Fill → Int → FlowDirection → Log2 → fldir_deg`.
   - **PART 2** — Per-AOI iteration: `Feature to Raster → Cost Allocation → Path Distance → Extract by Mask → z_diffse → pgse → filter by Tanα → save`.
4. Collect output rasters (`path_<ID>.tif`) from the output folder.

> **Note:** Requires the **Spatial Analyst** extension in ArcGIS Pro.  
> **Note:** If Tanα is left blank, a default value of `0.05` is used.

---

## License

See the [LICENSE](LICENSE) file for full terms.
