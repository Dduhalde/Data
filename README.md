# Dataset: GNSS-derived PWV and surface meteorology for precipitation nowcasting in continental Chile

This dataset contains the exact input data used in:

> Duhalde, D., Martinez-Villalobos, C., Valenzuela, R. A., & Campos, D. A. (2026). *Persistence captures much of the short-lead predictability of precipitation in Chile*. Manuscript in preparation.

It includes (i) hourly surface meteorological observations from 9 stations of the Dirección Meteorológica de Chile (DMC), (ii) hourly Zenith Total Delay (ZTD) from 9 collocated GNSS stations processed by the Nevada Geodetic Laboratory (NGL), (iii) 9 consolidated station files combining both sources with derived Precipitable Water Vapor (PWV), and (iv) ERA5 hourly single-level reanalysis fields over continental Chile.

**Version:** 1.0.0  **Contact:** Diego Duhalde (dduhalde@alumnos.uai.cl), ORCID: [0009-0001-1926-1978](https://orcid.org/0009-0001-1926-1978)

---

## 1. Directory structure

```
├── README.md
├── data_dictionary.csv
├── stations.csv
├── 01_raw/
│   ├── dmc/          9 × .parquet  (DMC surface meteorology, hourly)
│   ├── ngl/          9 × .parquet  (NGL ZTD, hourly)
│   └── era5/         ERA5_Chile.nc  (ERA5 single levels, continental Chile)
└── 02_processed/
    └── consolidated/ 9 × .parquet  (DMC + NGL + PWV, hourly, selected years)
```

File naming: `<DMC code>.parquet`, `<GNSS code>.parquet`, `<DMC code>_<GNSS code>.parquet`

In Zenodo, each data folder is provided as a separate zip archive (`01_raw_dmc.zip`, `01_raw_ngl.zip`, `01_raw_era5.zip`, `02_processed_consolidated.zip`). Extracting all archives into the same directory reproduces the structure shown above.

## 2. Stations

Nine DMC–GNSS station pairs were used. Coordinates and metadata are in `stations.csv`.

| DMC code | GNSS code | DMC lat | DMC lon | DMC height (m) | GNSS height (m) | Horizontal distance (km) | Height difference GNSS − DMC (m) |
|---|---|---|---|---|---|---|---|
| 290004 | LSCH | -29.91444 | -71.20667 | 146.0 | 78.831 | 3.86 | -67.2 |
| 320063 | CTPC | -32.56139 | -71.29778 | 81.0 | 104.653 | 0.05 | 23.7 |
| 330020 | DGF1 | -33.44500 | -70.68278 | 520.0 | 581.404 | 2.34 | 61.4 |
| 330030 | RCSD | -33.65611 | -71.61333 | 77.0 | 92.613 | 0.24 | 15.6 |
| 360019 | CONZ | -36.78055 | -73.06639 | 8.0 | 176.228 | 7.92 | 168.2 |
| 450004 | BN11 | -45.59083 | -72.10222 | 299.0 | 308.230 | 3.33 | 9.2 |
| 510005 | PNAT | -51.66722 | -72.52889 | 69.0 | 111.047 | 2.16 | 42.0 |
| 520012 | PARC | -53.13000 | -70.88750 | 11.0 | 22.307 | 0.93 | 11.3 |
| 550001 | PWIL | -54.93167 | -67.61556 | 12.0 | 36.010 | 0.92 | 24.0 |

Distances were computed with the haversine formula (R = 6371 km).

## 3. Data sources and provenance

### 3.1 DMC surface meteorology (`01_raw/dmc/`)
- **Provider:** Dirección Meteorológica de Chile (DMC), (https://climatologia.meteochile.gob.cl)
- **Downloaded:** March 2026
- **Variables:** air temperature (°C), relative humidity (%), pressure (hPa), rainfall (mm)
- **Temporal resolution:** hourly, derived from the original 1-minute series by hourly mean
- **Pressure type:** station-level pressure
- **Rainfall:** accumulated over the previous hour, ending at the timestamp

### 3.2 NGL Zenith Total Delay (`01_raw/ngl_ztd/`)
- **Provider:** Nevada Geodetic Laboratory, University of Nevada, Reno (http://geodesy.unr.edu)
- **Downloaded:** March 2026
- **Variable:** ZTD (mm)
- **Temporal resolution:** hourly, derived from the original 5-minute series by hourly mean
- **Time reference:** UTC

### 3.3 Consolidated station files (`02_processed/consolidated/`)
DMC and NGL series were merged on their hourly datetime index for each station pair, and PWV was derived from ZTD and surface meteorology as described in Section 4. For each station, only the 6 years with the highest data completeness are retained:

| Pair | Years retained |
|---|---|
| 290004–LSCH | [2018,2020,2021,2023,2024,2025] | 
| 320063–CTPC | [2018,2019,2020,2021,2023,2025] | 
| 330020–DGF1 | [2019,2020,2021,2023,2024,2025] | 
| 330030–RCSD | [2017,2018,2021,2023,2024,2025] | 
| 360019–CONZ | [2020,2021,2022,2023,2024,2025] | 
| 450004–BN11 | [2017,2018,2019,2020,2022,2023] | 
| 510005–PNAT | [2017,2018,2019,2020,2023,2024] | 
| 520012–PARC | [2017,2018,2019,2020,2022,2023] | 
| 550001–PWIL | [2018,2019,2020,2021,2024,2025] | 

### 3.4 ERA5 reanalysis (`01_raw/era5/`)
- **Provider:** Copernicus Climate Change Service (C3S) Climate Data Store
- **Dataset:** ERA5 hourly data on single levels from 1940 to present (DOI: 10.24381/cds.adbb2d47)
- **Downloaded:** April 2026
- **Domain:** continental Chile
- **Period:** 2017-01-01 00:00 to 2025-12-31 23:00 UTC, hourly (78,888 time steps)
- **Variables:**

| Name | Description | Units |
|---|---|---|
| t2m | 2 m temperature | K |
| d2m | 2 m dewpoint temperature | K |
| sp | Surface pressure | Pa |
| tp | Total precipitation | m |
| tcwv | Total column vertically integrated water vapour | kg m⁻² (≈ mm of PWV) |

## 4. PWV derivation

PWV was derived from ZTD following Bevis et al. (1992). All steps were applied at hourly resolution to each DMC–GNSS station pair.

1. **Zenith hydrostatic delay (ZHD):** Saastamoinen (1972) model with the Davis et al. (1985) gravity correction:

   ZHD = 2.2768 · P / (1 − 0.00266 · cos(2φ) − 0.00028 · H)

   where ZHD is in mm, P is the DMC surface pressure (hPa), φ is the station latitude and H is the station height (km). [TODO: confirm whether the H term was included, and whether P was used as measured at the DMC station or corrected to the GNSS antenna height.]

2. **Zenith wet delay:** ZWD = ZTD − ZHD (mm).

3. **Weighted mean temperature:** Tm = 70.2 + 0.72 · Ts (Bevis et al., 1992), with Ts the DMC surface air temperature converted to K.

4. **Conversion to PWV:** PWV = Π · ZWD, with

   Π = 10⁶ / [ρw · Rv · (k3/Tm + k2′)]

   Constants (Bevis et al., 1994): ρw = 1000 kg m⁻³, Rv = 461.5 J kg⁻¹ K⁻¹, k2′ = 0.221 K Pa⁻¹ (22.1 K hPa⁻¹), k3 = 3739 K² Pa⁻¹ (3.739 × 10⁵ K² hPa⁻¹). Π is dimensionless and ≈ 0.15 for typical Tm values.

Note: the PWV tendencies used as predictors in the paper (ΔPWV over 1 h and 3 h) are not stored in the consolidated files; they are computed on the fly by the modelling code from the `PWV (mm)` column.

## 5. File format

All station files are Apache Parquet, readable with e.g.:

```python
import pandas as pd
df = pd.read_parquet("02_processed/consolidated/<file>.parquet")
```

The ERA5 file is NetCDF, readable with `xarray.open_dataset()`. Column names, units and descriptions are listed in `data_dictionary.csv`.