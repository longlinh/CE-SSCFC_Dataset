# Landsat-8 land-cover benchmark for semi-supervised collaborative clustering: Hanoi, Thanh Hoa, Ho Chi Minh City, Hai Phong (Vietnam) and Valencia, Alicante (Spain)

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.22304965.svg)](https://doi.org/10.5281/zenodo.22304965)

**Archived on Zenodo:** concept DOI [10.5281/zenodo.22304965](https://doi.org/10.5281/zenodo.22304965) (all versions); version 1.1.0 DOI [10.5281/zenodo.22855907](https://doi.org/10.5281/zenodo.22855907). Licence CC BY 4.0.

Six Landsat-8 surface-reflectance scenes with independent land-cover reference maps derived from
ESA WorldCover 2021, prepared for benchmarking semi-supervised and collaborative (multi-site)
clustering / classification methods. Two sets share one acquisition protocol:

| Set | Sites | Bands | Classes | Folder |
|---|---|---|---|---|
| Vietnam (VN3 + HP) | Hanoi, Thanh Hoa, Ho Chi Minh City, Hai Phong | B2–B5 (4) | 5 | `landsat8/{hn,th2,hcm2,hp}/` |
| Spain (ES2) | Valencia, Alicante | B2–B7 (6, incl. SWIR) | 6 | `landsat8/{valencia,alicante}/` |

Version 1.1.0 adds Hai Phong (`hp`), a held-out site built with the same pipeline and added only after the
evaluated methods and their parameters had been fixed; version 1.0.0 (doi:10.5281/zenodo.22304966) contains the five original sites.

Labelled regions, validation folds and train/test partitions are regenerated at run time from the
reference maps (protocol below), so no split files are distributed.

## Acquisition and preprocessing (both sets)

| Property | Value |
|---|---|
| Satellite | Landsat 8 OLI/TIRS (NASA/USGS), Collection 2 Tier 1 Level-2 (`LANDSAT/LC08/C02/T1_L2`) |
| Scene filter | metadata cloud cover < 10 %, acquisition 2020-01-01 – 2023-12-31 |
| Pixel mask | QA_PIXEL bits 3 (cloud) and 4 (cloud shadow), CFMask |
| Reflectance | surface reflectance (LaSRC), scale factor 2.75e-5 and offset −0.2 applied |
| Composite | per-pixel median of all remaining scenes, per band |
| Platform | Google Earth Engine (`ee.Image.median()`, export `scale=50`) |
| Working grid | nominal 50 m (0.00045°, EPSG:4326 / WGS84); native 30 m pixels are **resampled by nearest neighbour** (GEE default), not averaged |
| Cloud gaps | no NoData pixel in any band of any site |

## Set 1 — Vietnam (VN3): four bands, five classes

| Region | Code | Size (W × H) | Pixels | Bounding box (lon / lat) |
|---|---|---|---|---|
| Hanoi | `hn` | 1201 × 1001 | 1,202,201 | 105.580–106.120°E / 20.850–21.300°N |
| Thanh Hoa | `th2` | 981 × 825 | 809,325 | 105.400–105.840°E / 19.740–20.110°N |
| Ho Chi Minh City | `hcm2` | 1336 × 1114 | 1,488,304 | 106.370–106.970°E / 10.675–11.175°N |
| Hai Phong | `hp` | 1204 × 1003 | 1,207,612 | 106.410–106.950°E / 20.625–21.075°N |
| **Total** | | | **4,707,442** | |

Reference map: the 10 m WorldCover map (Zanaga et al. 2022, doi:10.5281/zenodo.7254221, CC BY 4.0)
is aggregated to the 50 m grid by majority vote (`reduceResolution(mode)`), snapped to the Landsat
grid (nearest) and remapped to five classes: 80, 90 → Water; 50, 60 → Built-up; 30, 40 →
Agriculture; 20, 95, 100 → Forest; 70 → NoData. Tree cover (10) is split by the NDVI of the Landsat
composite (threshold 0.55) into Perennial vegetation (< 0.55) and Forest (≥ 0.55).

| Value | Class | Hanoi | Thanh Hoa | Ho Chi Minh City | Hai Phong |
|---|---|---|---|---|---|
| 0 | Water | 4.39 % | 2.67 % | 4.09 % | 27.99 % |
| 1 | Built-up | 26.58 % | 9.25 % | 30.07 % | 15.11 % |
| 2 | Agriculture | 44.74 % | 44.84 % | 29.06 % | 32.51 % |
| 3 | Perennial vegetation | 11.30 % | 5.35 % | 9.78 % | 6.89 % |
| 4 | Forest | 12.99 % | 37.89 % | 27.00 % | 17.50 % |

Files per region (`{r}` ∈ hn, th2, hcm2, hp):

```
landsat8/{r}/
  {r}_z50_SR_B2.tif … {r}_z50_SR_B5.tif        Float32 surface reflectance (blue, green, red, NIR)
  {r}_z50_RGB.tif                              UInt8 composite (B4, B3, B2)
  {r}_z50_WorldCover2021_5class.tif            reference map, UInt8 0–4, 255 = NoData
  {r}_z50_WorldCover2021_5class_rgb.tif        colour-coded reference map
  {r}_z50_gt_validation.png                    visual check: RGB | reference overlay
```

Suggested protocol: class-pure circular labelled regions of radius 30 px, `n_regions_per_class`
= 10 / 14 / 24 / 10 for hn / th2 / hcm2 / hp (≈ 3.8 % / 7.2 % / 7.2 % / 5.0 % of pixels); spatially disjoint hold-out
by 32 × 32-pixel blocks (≈ 30 % of blocks, labelled pixels always in the training partition);
features z-scored per site.

## Set 2 — Spain (ES2): six bands, six classes

| Region | Code | Size (W × H) | Pixels | Bounding box (lon / lat) |
|---|---|---|---|---|
| Valencia | `valencia` | 1201 × 1002 | 1,203,402 | 0.160–0.700°W / 39.100–39.550°N |
| Alicante | `alicante` | 1201 × 1002 | 1,203,402 | 0.430–0.970°W / 38.025–38.475°N |
| **Total** | | | **2,406,804** | |

Reference map: WorldCover 2021 v200 aggregated 10 → 50 m by majority vote, snapped (nearest),
remapped without any NDVI split: 10, 95, 100 → Tree cover; 20 → Shrubland; 30 → Grassland;
40 → Cropland; 50, 60 → Built-up; 80, 90 → Water; 70 → NoData (absent).

| Value | Class | Valencia | Alicante |
|---|---|---|---|
| 0 | Tree cover | 22.13 % | 10.05 % |
| 1 | Shrubland | 10.33 % | 10.01 % |
| 2 | Grassland | 17.32 % | 35.98 % |
| 3 | Cropland | 14.21 % | 8.66 % |
| 4 | Built-up | 10.04 % | 13.62 % |
| 5 | Water | 25.98 % | 21.68 % |

Files per region (`{r}` ∈ valencia, alicante):

```
landsat8/{r}/
  {r}_z50_SR_B2.tif … {r}_z50_SR_B7.tif        Float32 surface reflectance (blue … SWIR-2)
  {r}_z50_RGB.tif                              UInt8 composite (B4, B3, B2)
  {r}_z50_WorldCover2021_6class.tif            reference map, UInt8 0–5, 255 = NoData
```

Suggested protocol: circular labelled regions per class with Valencia as the label-rich site
(16 regions, radius 30 px, ≈ 10.8 %) and Alicante as the label-poor site (12 regions, radius 15 px,
≈ 2.0 %); 32 × 32-block hold-out ≈ 30 %; z-score per site on the six bands.

## Loading example

```python
import rasterio, numpy as np
r, bands = "hn", (2, 3, 4, 5)            # or r, bands = "valencia", (2, 3, 4, 5, 6, 7)
X = np.stack([rasterio.open(f"landsat8/{r}/{r}_z50_SR_B{b}.tif").read(1) for b in bands], -1)
gt = "WorldCover2021_5class" if r in ("hn", "th2", "hcm2") else "WorldCover2021_6class"
y = rasterio.open(f"landsat8/{r}/{r}_z50_{gt}.tif").read(1)   # 255 = NoData
X, y = X.reshape(-1, len(bands)), y.reshape(-1)
```

`MD5SUMS` lists the checksum of every data file.

## Licence and citation

- Landsat-8 imagery: courtesy of the U.S. Geological Survey, public domain.
- Reference maps derived from ESA WorldCover 2021 v200 (CC BY 4.0) — cite Zanaga et al. (2022),
  doi:10.5281/zenodo.7254221. Contains modified Copernicus Sentinel data (2021).
- Derived products in this repository: CC BY 4.0 (see `LICENSE`). Cite the Zenodo record
  (see `CITATION.cff`): X. H. Nguyen, *Landsat-8 land-cover benchmark for semi-supervised collaborative clustering: Hanoi, Thanh Hoa, Ho Chi Minh City, Hai Phong (Vietnam) and Valencia, Alicante (Spain)*, version 1.1.0, Zenodo, 2026, doi:10.5281/zenodo.22855907.

Related publications: listed in the Zenodo record metadata and updated as papers using this data
appear.
