# Hurricane Melissa Damage Assessment — Jamaica

Multi-indicator satellite remote sensing pipeline for quantifying hurricane damage across five urban areas in Jamaica, built in Google Earth Engine / Python following Hurricane Melissa (Category 5, October 28, 2025).

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/GarretMaloney/DamageAssessment/blob/main/Maloney_Final_Project.ipynb)

## Overview

Hurricane Melissa struck Jamaica on October 28, 2025, with the most severe reported impacts concentrated in the west (Montego Bay, Falmouth). This project fuses four independent satellite-based damage indicators into a per-region damage score for five urban areas — Kingston, Montego Bay, Falmouth, Negril, and Ocho Rios — and critically evaluates whether that score actually tracks reported ground-truth severity.

## Methodology

Four complementary indicators, computed in Google Earth Engine from Sentinel-1 SAR and Sentinel-2 optical imagery:

| Indicator | Data | What it captures | Threshold |
|---|---|---|---|
| **SAR coherence** | Sentinel-1 (pre/post pair) | Structural collapse, per building | mean < 0.5, or any pixel < 0.3 -> damaged |
| **Backscatter decrease** | Sentinel-1 | Flooding / standing water | < -2 dB |
| **Backscatter increase** | Sentinel-1 | Wind damage / debris accumulation | > +2 dB |
| **NDVI change** | Sentinel-2 | Vegetation loss | decrease > 0.15 |

Per-building structural damage is assessed against Microsoft Building Footprints rather than raw pixels. An automated image-pair selection step balances two competing constraints — temporal proximity to the hurricane (<=21 days) and spatial coverage of the area of interest (>80%) — since satellite swaths from the same pass can otherwise leave large coverage gaps.

Indicators are combined into a 0-40 point weighted damage score (10 points each for structural, water, wind, and vegetation damage) and normalized to a 0-100% severity scale, with several alternative weighting schemes evaluated (urban/infrastructure-weighted, environmental-weighted, severity-scaled).

## Results

| Region | Score (0-40) | Severity | Buildings damaged |
|---|---|---|---|
| Kingston | 15.94 | 39.9% | 61,966 |
| Falmouth | 15.80 | 39.5% | 817 (54% vegetation loss) |
| Negril | 14.21 | 35.5% | 1,299 |
| Montego Bay | 14.18 | 35.5% | 10,072 (50% vegetation loss) |
| Ocho Rios | 11.74 | 29.4% | 1,996 |

## Key Finding: The Score Disagrees With Ground Truth — And That's the Point

News reports identified Montego Bay and Falmouth as the most severely damaged areas. The point-based score ranks them 4th and 2nd, with Kingston first. Rather than treating this as a bug to paper over, the notebook diagnoses *why*:

- **Building count dominates the structural score.** Kingston's 61,966 buildings vs. Falmouth's 817 means Kingston's structural score reflects urban density, not damage intensity — a building coherence of 0.65 (minor) and 0.3 (destroyed) both just count as "damaged."
- **Polygon size compounds the bias.** Normalizing by area (points/km²) instead of using absolute score reverses the picture: Falmouth's damage density is roughly 9.6x Kingston's.
- **Vegetation loss is a better proxy here.** Re-ranking by vegetation loss alone puts Falmouth (54%) and Montego Bay (50%) at the top — consistent with reported severity — suggesting environmental damage may be a more reliable signal than building-based scoring for coastal/resort areas in a Category 5 event.

This distinction — extent vs. severity, and count-based vs. density-based scoring — is the main methodological takeaway of the project.

## Limitations

- Three of four indicators depend on Sentinel-1 alone (single-sensor dependency)
- Damage thresholds (coherence, dB, NDVI) are physically motivated but not validated against field/ground-truth surveys
- Polygon (AOI) definition materially affects results; no standardized urban boundary dataset was used
- SAR coherence cannot detect flooding (water is SAR-smooth) or non-collapse damage (roof/interior damage on a standing building)
- Cloud cover limits Sentinel-2/NDVI availability, a common post-hurricane constraint

## Stack

Google Earth Engine (`ee`, `geemap`), Sentinel-1 GRD, Sentinel-2, Microsoft Building Footprints, Python/NumPy, Google Colab.

## Notebook

[`Maloney_Final_Project.ipynb`](Maloney_Final_Project.ipynb) — full pipeline: AOI definition -> image-pair selection -> per-indicator damage computation -> per-building zonal statistics -> weighted scoring -> interactive map visualization -> results/limitations analysis.

## Development Notes

This project was built for a remote sensing course that encouraged AI-assisted coding as long as its use was documented and critically reflected on. AI helped iterate on the pipeline quickly and proposed the backscatter-based water/wind damage indicators as an extension beyond the original building-and-vegetation scope.

The clearest lesson came from a coverage bug: Ocho Rios' structural damage layer only covered a small sliver of the area despite full NDVI coverage. Initial attempts to have AI patch the coherence calculation directly failed to fix it. Asking for a root-cause diagnosis instead of a fix revealed the real issue — the pre/post Sentinel-1 image pair only overlapped 6% spatially — which led to building the automated image-pair selection logic (balancing temporal proximity and spatial coverage) used throughout the rest of the pipeline. The general takeaway: ask for diagnosis before asking for a fix, and keep decisions about scope and interpretation human-driven.
