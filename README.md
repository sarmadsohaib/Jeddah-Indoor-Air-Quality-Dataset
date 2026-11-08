# Jeddah Indoor Air Quality Dataset — Sites 1 and 5

## Overview

This repository contains time-series indoor air quality (IAQ) measurements collected at **two residential indoor monitoring sites in Jeddah, Saudi Arabia**. The records are a publicly shared **two-site subset of a broader eight-site study** investigating short-term, multivariate IAQ forecasting using LSTM, BiLSTM, and Transformer–BiLSTM models.

The measurements cover carbon dioxide (CO₂), temperature, relative humidity, formaldehyde (HCHO), VOC and NOₓ indices, airborne particulate matter, particle counts, and a binary presence signal. The sensing system used commercially available environmental sensors integrated into a custom IoT monitoring device.

**Important scope note:** Sites 1 and 5 were reserved as **held-out test sites** in the associated study. The remaining six sites used for model development are **not included** in this repository. Accordingly, this two-site release can support independent analysis and testing, but **does not by itself allow full reproduction of the model training reported in the paper**.

## Dataset files

| File | Site | Number of records | First timestamp | Last timestamp | Site description |
|---|---|---:|---|---|---|
| [`Site-1_dataset(4).csv`](<Site-1_dataset(4).csv>) | Site 1 | 27,487 | 2026-01-13 17:22:53 | 2026-01-23 17:59:28 | Residential indoor environment with relatively limited natural ventilation |
| [`Site-5_dataset(20261108-084710).csv`](<Site-5_dataset(20261108-084710).csv>) | Site 5 | 35,180 | 2026-04-07 13:18:10 | 2026-04-20 01:04:02 | Residential indoor environment with more frequent window opening / natural ventilation |
| **Total** | **2 sites** | **62,667** | — | — | — |

Both files contain **23 columns**, including the `date` and `time` fields and 21 measurement/indicator columns.

### Time and sampling information

- The intended sampling interval was approximately **30 seconds**. The median interval between consecutive observations is **31 seconds** in both uploaded files; measurements are not precisely evenly spaced.
- **Date formats differ between files:** Site 1 uses `M/D/YYYY` (for example, `1/13/2026`); Site 5 uses `YYYY-MM-DD` (for example, `2026-04-07`). The `time` column uses `HH:MM:SS` (24-hour clock).
- Timestamps do not include UTC offsets or an explicit time-zone identifier in the CSV. Treat them as **timezone-naive** until the acquisition time zone is confirmed.
- The supplied files contain no blank/null CSV fields, but **a nonblank value is not necessarily a valid measurement**. For example, zero-valued gas indices at the start of a recording may require examination for initialization or warm-up effects.
- A few larger gaps occur between observations. Account for actual timestamps rather than assuming a perfectly uniform time axis.

## Measurement hardware

| Sensor / signal source | Variables in this release | Measurement principle / role |
|---|---|---|
| **SCD30** | CO₂, temperature, relative humidity | NDIR CO₂ sensor with temperature and humidity measurements |
| **SGP41** | Raw VOC/NOₓ signals, VOC Index, NOₓ Index | Metal-oxide-based gas sensing with processed relative air-quality indices |
| **SPS30** | PM mass concentrations, particle number concentrations, typical particle size | Laser-based particulate matter sensing |
| **SFA30** | HCHO, temperature, relative humidity | Electrochemical formaldehyde sensing with environmental measurements |
| **Presence signal** | Binary presence output | Presence-related indicator recorded by the monitoring device; the exact sensing/firmware logic should be verified before use |

The SGP41 **VOC Index and NOₓ Index are relative indicators, not absolute VOC or NOₓ concentrations**. The SFA30 formaldehyde measurement is a separate gas-specific measurement; formaldehyde is itself a VOC.

## Data dictionary

The column names below match the supplied CSV files exactly.

| CSV column | Description | Units / interpretation |
|---|---|---|
| `date` | Recording date | Site-specific text date format (see above) |
| `time` | Recording time | `HH:MM:SS` |
| `presence` | Binary presence indicator | `0` or `1`; verify the firmware's detection definition |
| `sgp41_srawVOC` | SGP41 raw VOC-sensitive signal | Digital raw ticks; **not** ppm or ppb |
| `sgp41_srawNOx` | SGP41 raw NOₓ-sensitive signal | Digital raw ticks; **not** ppm or ppb |
| `sgp41_vocIndex` | Processed VOC Index | Unitless index points; relative indicator |
| `sgp41_noxIndex` | Processed NOₓ Index | Unitless index points; relative indicator |
| `scd30_co2_ppm` | Carbon dioxide | ppm |
| `scd30_temp_c` | Temperature measured by SCD30 | °C |
| `scd30_rh` | Relative humidity measured by SCD30 | % RH |
| `sps30_mc1.0` | PM₁.₀ mass concentration | µg/m³ (sensor-standard unit) |
| `sps30_mc2.5` | PM₂.₅ mass concentration | µg/m³ (sensor-standard unit) |
| `sps30_mc4.0` | PM₄.₀ mass concentration | µg/m³ (sensor-standard unit) |
| `sps30_mc10` | PM₁₀ mass concentration | µg/m³ (sensor-standard unit) |
| `sps30_nc0.5` | Particle number concentration, 0.5 µm size channel | particles/cm³ (sensor-standard unit) |
| `sps30_nc1.0` | Particle number concentration, 1.0 µm size channel | particles/cm³ (sensor-standard unit) |
| `sps30_nc2.5` | Particle number concentration, 2.5 µm size channel | particles/cm³ (sensor-standard unit) |
| `sps30_nc4.0` | Particle number concentration, 4.0 µm size channel | particles/cm³ (sensor-standard unit) |
| `sps30_nc10` | Particle number concentration, 10 µm size channel | particles/cm³ (sensor-standard unit) |
| `sps30_tps` | SPS30 typical particle size (TPS) | **Logged as an integer**; confirm scale/unit in device firmware before converting (SPS30 integer-format output uses nm) |
| `sfa30_hcho` | Formaldehyde (HCHO) concentration | ppb (sensor-standard unit) |
| `sfa30_rh` | Relative humidity measured by SFA30 | % RH |
| `sfa30_temp_c` | Temperature measured by SFA30 | °C |

**Unit verification:** The listed sensor-standard units are based on manufacturer documentation. This README was prepared from the CSV contents and study description, not the original acquisition firmware. Verify the logged-unit conversions—particularly `sps30_tps`—against the firmware before interpreting physical magnitudes.

## Quick start (Python)

Requires Python 3 and `pandas` (`pip install pandas`). This example reads both CSV files and constructs timestamps using each file's actual date format.

```python
import pandas as pd

site1 = pd.read_csv("Site-1_dataset(4).csv")
site5 = pd.read_csv("Site-5_dataset(20261108-084710).csv")

site1["timestamp"] = pd.to_datetime(
    site1["date"] + " " + site1["time"],
    format="%m/%d/%Y %H:%M:%S",
)
site5["timestamp"] = pd.to_datetime(
    site5["date"] + " " + site5["time"],
    format="%Y-%m-%d %H:%M:%S",
)

site1["site"] = "Site 1"
site5["site"] = "Site 5"

combined = pd.concat([site1, site5], ignore_index=True)
combined = combined.sort_values(["site", "timestamp"])

print("Rows:", len(combined))
print(combined.groupby("site")["scd30_co2_ppm"].describe())
```

**Forecasting caution:** Split observations by site and time **before** fitting any imputation, scaling, PCA, or model parameters. Avoid distributing highly overlapping sequence windows across training and test sets, as this can produce information leakage.

## Data quality and limitations

- The two site files were examined for column names, date ranges, row counts, and empty fields. They have not been independently validated against reference-grade instruments.
- The study describes use of manufacturer-supplied calibration and compensation features, but **no independent laboratory calibration** was reported.
- Sensor drift, environmental interferences, warm-up periods, irregular sampling, and possible outliers should be considered.
- SGP41 index values should not be interpreted as gas concentration measurements.
- The data are from **two indoor environments** and do not represent all buildings or seasons in Jeddah.
- This subset contains **test sites only** from the broader eight-site study; training datasets, training code, and model weights are not included here.

## Associated publication and how to cite

This dataset is associated with the manuscript:

> **Transformer BiLSTM-Based Multivariate Indoor Air Quality Forecasting for Edge IoT Smart Buildings**. *PLOS ONE* (publication details and DOI to be inserted after publication).

**If you use these data in a publication, please cite the associated paper when it becomes available and cite the dataset repository/release itself.** For a stable dataset citation, archive a numbered GitHub release in [Zenodo](https://zenodo.org/) and add the assigned dataset DOI below.

- **Paper DOI:** `To be added after publication`
- **Dataset DOI:** `To be added after archiving a release`
- **Dataset authors and citation year:** `To be confirmed by the dataset creators`

Please do not use the manuscript title as if the article has already been published before its publication details are confirmed.

## Access permissions, privacy, and reuse

Site access for the field study was obtained through verbal permission from the owners or authorized occupants; no official field-site permits were required according to the study description.

**Before making this repository public**, confirm that permission extends to **public dissemination** of these measurements. In particular, the files contain timestamped `presence` records, which may reveal patterns of occupancy. Consider removing or aggregating that field, or adjusting date/time granularity if this is necessary to protect participants' privacy, and ensure any applicable research-ethics obligations are satisfied.

**License:** Not yet specified. Add a data-reuse license (for example, CC BY 4.0, if all rights holders agree) before publication. A citation request in a README does not replace an actual license.

## Sensor documentation

- [Sensirion SCD30](https://sensirion.com/products/catalog/SCD30)
- [Sensirion SGP41](https://sensirion.com/resource/datasheet/sgp41)
- [Sensirion SPS30 datasheet](https://sensirion.com/resource/datasheet/sps30)
- [Sensirion SFA30 datasheet](https://sensirion.com/media/documents/DEB1C6D6/63D92360/Sensirion_formaldehyde_sensors_datasheet_SFA30.pdf)

## Contact

For dataset questions or corrections, please use the repository's **Issues** section. A corresponding-author contact may be added by the dataset maintainers.
