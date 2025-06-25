# 📌 Assessing Relationships between Land Use Diversity and Vegetation Resilience during Heatwaves in Brandenburg, Germany

**Author:** *Konlavach Mengsuwan*  
**Affiliation:** *ZALF, AIA Working Group*  
**Date:** *June 2025*

![GIF1](https://github.com/user-attachments/assets/944e8209-5cc4-4d27-b0d2-37dcd583566e)

---

## 🌱 Introduction

The resilience of terrestrial ecosystems to climatic extremes, particularly heatwaves, is influenced significantly by land-use practices and landscape diversity. Landscapes with higher diversity often support greater biodiversity and potentially increased resilience to climate-induced disturbances. This study investigates the relationship between landscape heterogeneity (as quantified by land-use diversity) and vegetation resilience during a heatwave event in Brandenburg, Germany.

We specifically focus on the **2022 summer heatwave event** in Brandenburg and analyze vegetation resilience using remotely-sensed vegetation indices (NDVI from Sentinel-2 imagery). The Shannon diversity index derived from CORINE land cover data represents landscape diversity at a spatial scale of 10 km × 10 km.

---

## 🎯 Objectives

The key objectives of this study are to:

1. **Quantify land-use diversity** at the landscape scale (10 km × 10 km grid cells) using the Shannon diversity index based on CORINE land cover data.
2. **Assess vegetation resilience** by quantifying the NDVI drop (June vs. July–August heatwave period) derived from Sentinel-2 satellite imagery.
3. **Examine and interpret the relationship** between land-use diversity and vegetation resilience under heatwave conditions.

---

## 🛠️ Methods

The analysis was performed using the Google Earth Engine (GEE) platform. The following workflow summarizes the key steps:

### ✅ Step 1: Grid Creation

A precise 10 km × 10 km grid was generated using a raster-slicing method with a random image aligned to UTM Zone 33N projection (`EPSG:32633`):

![image](https://github.com/user-attachments/assets/807d2e30-0f9f-4a82-95f8-7536b91d5f47)

```javascript
var projection = ee.Projection("EPSG:32633");
var dummy = ee.Image.random().reproject(projection, null, 10000);
var tiled = dummy.multiply(10000).toInt();

var grid = tiled.reduceToVectors({
  geometry: brandenburg.geometry(),
  scale: 10000,
  geometryType: "polygon",
  reducer: ee.Reducer.countEvery(),
  maxPixels: 1e10
}).filterBounds(brandenburg);
```

### ✅ Step 2: Shannon Diversity Index Calculation

The CORINE Land Cover dataset (2018) was used to compute the **Shannon Diversity Index** for each 10 km × 10 km grid cell. CORINE data were clipped to the Brandenburg boundary and reprojected to match the grid projection (`EPSG:32633`).

A frequency histogram of land cover classes was generated per grid cell using Earth Engine’s `frequencyHistogram()` reducer. Then, the Shannon index was calculated using the following logic:

```javascript
var shannon = hist.values().map(function(val) {
  val = ee.Number(val);
  var p = val.divide(total);
  return p.multiply(p.log()).multiply(-1);
}).reduce(ee.Reducer.sum());
```

Where:

- **val** is the count of pixels for each land cover class.
- **p** is the proportion of each class.

The **Shannon Diversity Index (H′)** is computed as:

&nbsp;&nbsp;&nbsp;&nbsp;*H′ = −∑ (pᵢ × ln(pᵢ))*

This index reflects the degree of land use diversity. A higher H′ value indicates more heterogeneous land use within a grid cell.

---

### ✅ Step 3: NDVI Drop Calculation

To measure vegetation stress during heatwaves, we calculated the difference in NDVI between two time periods:

![image](https://github.com/user-attachments/assets/a98e9d45-e2ab-4834-8673-b329be28bff9)

- **Pre-heatwave period:** June 1–30, 2022  
- **Heatwave period:** July 15 – August 15, 2022

NDVI was calculated from Sentinel-2 imagery using the normalized difference between near-infrared (B8) and red (B4) bands:

&nbsp;&nbsp;&nbsp;&nbsp;**NDVI = (B8 − B4) / (B8 + B4)**

The NDVI values were averaged for each period, and the **NDVI drop** was computed as:

&nbsp;&nbsp;&nbsp;&nbsp;**NDVI Drop = NDVI<sub>June</sub> − NDVI<sub>Heatwave</sub>**

This drop indicates vegetation vulnerability under heat stress:
- Larger drops suggest reduced greenness or productivity.
- Smaller drops suggest higher resilience.

---

### ✅ Step 4: Statistical Analysis

Each 10 km grid cell now contains:
- A **Shannon Diversity Index** value (landscape diversity)
- An **NDVI Drop** value (vegetation resilience)

To explore the relationship, we created a scatter plot of NDVI drop vs. Shannon diversity, with a linear regression trendline and R² value to quantify the correlation.

The analysis was done using Earth Engine’s `ui.Chart.feature.byFeature`, and results are visualized directly in the GEE environment.

---

## 📊 Results

The scatter plot below illustrates the relationship between land-use diversity (Shannon Index) and vegetation resilience (NDVI Drop) during the 2022 summer heatwave in Brandenburg:

![image](https://github.com/user-attachments/assets/ede8445c-4eb5-4f08-ae66-c24e842d86c5)

**Figure:** *Scatter plot showing the relationship between Shannon Diversity Index and NDVI Drop in Brandenburg, Germany (2022 heatwave period). Each point represents a 10 km × 10 km grid cell.*

**Key Findings:**

- Grid cells with higher **Shannon diversity** generally showed **smaller NDVI drops**.
- The fitted regression line suggests a **negative correlation** between land-use diversity and vegetation vulnerability.
- **Coefficient of determination (R²):** *0.233*, indicating moderate explanatory power.

This supports the hypothesis that landscape diversity contributes to vegetation stability under heat stress.

---

## 🗣️ Discussion

The results suggest that heterogeneous landscapes with greater land-use diversity may buffer ecosystems against extreme climatic events such as heatwaves. Diverse landscapes may support:
- Microclimatic regulation
- Resource and habitat heterogeneity
- Functional redundancy in vegetation types

These characteristics can enhance ecosystem resilience by maintaining vegetation productivity during stress periods.

However, the moderate R² value (0.233) indicates that diversity alone does not fully explain variation in NDVI response. Additional factors likely influence vegetation resilience, including:
- Soil type and water retention capacity
- Vegetation structure and rooting depth
- Land management practices (e.g., irrigation, tillage)
- Local topography and shading

This analysis provides a spatial foundation for integrating such variables in future resilience assessments.

---

## 📌 Limitations and Future Directions

**Limitations:**

- CORINE Land Cover has a 100 m resolution and may generalize heterogeneous land parcels.
- Sentinel-2 NDVI estimates are affected by cloud masking and temporal coverage.
- NDVI alone may not capture all aspects of vegetation function (e.g., phenology, biomass, or transpiration).

**Future Directions:**

- Incorporate multi-temporal NDVI trends and additional vegetation indices (e.g., EVI, LAI).
- Use finer-scale land cover or field-level management data.
- Integrate soil moisture, MODIS LST, or socio-ecological variables into multivariate models.
- Extend the framework to compare across years or regions.

---

## 💻 Repository Structure

```text
project-root/
│
├── README.md
├── GEE_script.js
├── scatter_plot.png
└── data/
    └── exported_grid_results.csv
```

## 🔗 Data Sources

- **CORINE Land Cover (2018):**  
  [https://land.copernicus.eu/pan-european/corine-land-cover](https://land.copernicus.eu/pan-european/corine-land-cover)

- **Sentinel-2 Surface Reflectance (Level-2A):**  
  [https://sentinel.esa.int/web/sentinel/missions/sentinel-2](https://sentinel.esa.int/web/sentinel/missions/sentinel-2)

- **Brandenburg Boundary — GAUL Level 1 (FAO):**  
  [https://developers.google.com/earth-engine/datasets/catalog/FAO_GAUL_2015_level1](https://developers.google.com/earth-engine/datasets/catalog/FAO_GAUL_2015_level1)

- **Google Earth Engine:**  
  [https://earthengine.google.com](https://earthengine.google.com)

---

GEE Code: https://code.earthengine.google.com/fc4e742fb26264691ef5b39a345a7174

## 📚 References

- Foley, J.A., et al. (2005). Global consequences of land use. *Science, 309*(5734), 570–574.  
- Turner, M.G., Gardner, R.H., & O'Neill, R.V. (2001). *Landscape Ecology in Theory and Practice*. Springer-Verlag, New York.  
- Copernicus Land Monitoring Service. CORINE Land Cover. [https://land.copernicus.eu](https://land.copernicus.eu)

---

## 📝 Citation

If you use or reference this repository in your work, please cite:

```bibtex
@misc{Mengsuwan2025landuse,
  author = {Mengsuwan, Konlavach},
  title = {Assessing Relationships between Land Use Diversity and Vegetation Resilience during Heatwaves in Brandenburg, Germany},
  year = {2025},
  url = {https://github.com/KonlavachMengsuwan/Assessing-Relationships-between-Land-Use-Diversity-and-Vegetation-Resilience-during-Heatwaves}
}

![image](https://github.com/user-attachments/assets/129e5540-5fc8-494c-b73f-d16edc0e9eca)

