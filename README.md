# Satellite-Based Remote Sensing Damage Assessment of Five Urban Areas in Jamaica

*Final project for a Remote Sensing course.*

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/GarretMaloney/DamageAssessment/blob/main/Maloney_Final_Project.ipynb)

Hurricane Melissa struck Jamaica on October 28, 2025, as one of the strongest Category 5 hurricanes on record, causing extensive damage across the island with the most severe impacts concentrated in the west. This project applies remote sensing techniques to quantify different forms of damage — structural, vegetation, debris, and flooding — across five urban areas.

## Scope Development

Initially, the plan was to perform a building damage assessment and quantify vegetation damage through NDVI analysis. During development, it became clear that building damage can occur without structural collapse — flooding and wind can ruin foundations or blow out windows while leaving buildings standing. Backscatter was added as a means of detecting flooded areas (lower backscatter) and debris-strewn areas (higher backscatter), giving two additional metrics for quantifying damage remotely. A combined damage index was also created to help assess overall severity.

## Methods

**Area of Interest.** Five areas were investigated: Kingston, Negril, Montego Bay, Falmouth, and Ocho Rios, selected to capture regional variation in damage. News reports indicated that Montego Bay and Falmouth experienced the most severe impacts, while other areas sustained damage to lesser degrees. Polygons were drawn around each area to approximate the urban extent visible in satellite imagery, then clipped to Jamaica's administrative boundary to define clean coastlines and exclude ocean areas from analysis.

**Building Damage.** Microsoft Building Footprints were imported from Google Earth Engine and filtered to each area of interest. Per-building damage assessment used SAR coherence calculated from pre- and post-hurricane Sentinel-1 imagery. Buildings were flagged as damaged if mean coherence fell below 0.7 (roof damage or structural issues) or if any pixel within the building showed coherence below 0.5 (severe damage).

**Backscatter Change.** Backscatter change analysis identified flooding and wind damage signatures. Pixels with backscatter decreases exceeding 2 dB were classified as flooded, with severity scaled by the magnitude of decrease. Backscatter increases exceeding 2 dB indicated debris accumulation from wind damage, similarly scaled by magnitude.

**NDVI.** Vegetation damage assessment used NDVI change from Sentinel-2 optical imagery. Unlike the other indicators, which rely on Sentinel-1 SAR data, this optical-based approach provides an independent data stream, reducing single-sensor bias. Pixels with NDVI decreases exceeding 0.15 were classified as experiencing significant vegetation loss.

**Combined Damage Assessment.** A combined damage index synthesized all four indicators at the pixel level:
- Maximum damage extent — pixels flagged by at least one indicator (useful for survey planning)
- High-confidence damage zones — pixels flagged by two or more indicators (priority areas for response)
- A continuous severity layer, on the assumption that pixels showing multiple damage signatures indicate more severe impacts

## Results

Point-based scoring (0–40 points: 10 each for structural, water, wind, and vegetation):

1. **Kingston**: 15.94/40 (39.9%) — 61,966 buildings damaged
2. **Falmouth**: 15.80/40 (39.5%) — 817 buildings damaged, 54% vegetation loss
3. **Negril**: 14.21/40 (35.5%) — 1,299 buildings damaged
4. **Montego Bay**: 14.18/40 (35.5%) — 10,072 buildings damaged, 50% vegetation loss
5. **Ocho Rios**: 11.74/40 (29.4%) — 1,996 buildings damaged

**Important caveats:** these rankings should be interpreted cautiously. Montego Bay and Falmouth — reported as severely damaged — rank 4th and 2nd, while Kingston ranks first. Kingston's top ranking may be an artifact of its large urban area and high building count rather than damage intensity: Kingston's 61,966 buildings vs. Falmouth's 817 means building count dominates the structural score regardless of per-building severity. No context-specific weighting was applied (urban vs. coastal areas treated equally), and damage density (points/km²) was not considered — normalizing by area, Falmouth's damage density is roughly 9.6x Kingston's.

Re-ranking by vegetation loss alone places Falmouth (54.0%) and Montego Bay (49.8%) at the top, aligning better with news reports of severe damage, ahead of Ocho Rios (34.0%), Negril (31.1%), and Kingston (15.8%).

## Shortcomings

**Polygon Definition.** Polygon definition significantly impacts damage assessments. Including less-developed areas with extensive vegetation led to inflated vegetation damage statistics, which can skew overall rankings. Future work should use standardized urban boundaries (e.g., Global Human Settlement Layer) for consistent area definitions.

**Temporal Resolution.** Temporal resolution posed challenges, particularly for optical imagery where cloud cover limited availability. The ephemeral nature of flooding means delays between hurricane landfall and post-event imaging can miss significant impacts — standing water recedes within days, though residual damage signatures remain detectable through backscatter and NDVI changes, which serve as proxies requiring careful threshold calibration.

**Single-Sensor Dependency.** Three of the four damage indicators rely on Sentinel-1 SAR data. While these indicators measure different physical phenomena (coherence vs. backscatter magnitude and direction), issues with Sentinel-1 calibration or processing could systematically bias results.

**Threshold and Weight Validation.** Current thresholds were iteratively refined through reasoning about physical implications but lack validation against ground-truth damage assessments. The damage weighting scheme similarly lacks statistical validation — the relative importance of structural vs. vegetation damage in determining overall severity requires domain expertise and empirical calibration against historical hurricane data.

## Reflections on AI-Assisted Development

This project was completed for a remote sensing course that encouraged AI-assisted coding as long as its use was documented and critically reflected on. AI is a powerful tool for iterating on code efficiently, but it requires critical oversight — a thorough understanding of the data, methods, and project scope is essential for recognizing when AI actions diverge from the actual objectives. Conversely, AI frequently proposed valuable extensions beyond the initial scope, such as the backscatter-based water and wind damage indicators. AI generated diagnostic tools were helpful in determining root cause failures that led to more appropriate fixes than having AI try to write a workaround to an inadequately defined error.

## Data Sources

Sentinel-1 SAR (C-band, VV polarization, IW mode), Sentinel-2 optical (NIR + Red for NDVI), Microsoft Building Footprints, custom AOI polygons clipped to Jamaica's administrative boundary. 10m resolution. Pre-event window: Oct 1–27, 2025; post-event window: Oct 29 – Nov 17, 2025; target temporal baseline ≤21 days.

## Notebook

[`Maloney_Final_Project.ipynb`](Maloney_Final_Project.ipynb)
