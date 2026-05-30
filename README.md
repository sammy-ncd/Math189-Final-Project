# Math 189 Final Project

**Research Question:** For San Francisco from 2023 to 2025, did the expansion of Waymo's driverless ride-hailing service correspond with changes in taxi activity, public transit ridership, and the broader transportation market?

---

## Project Structure

```
Math189-Final-Project/
├── data/
│   ├── passenger-miles-traveled-self-driving-taxis.csv  # Raw: Waymo/AV activity
│   ├── sf-taxi.csv                                       # Raw: SF taxi trips (sample)
│   ├── sf_taxi_monthly.csv                               # Raw: SF taxi trips (monthly)
│   ├── muni_ridership_monthly.csv                        # Raw: Muni ridership
│   ├── bart_monthly_exits.csv                            # Raw: BART exits
│   └── 02-processed/
│       ├── waymo_clean.csv                               # Cleaned: Waymo
│       ├── taxi_clean.csv                                # Cleaned: Taxi
│       ├── muni_clean.csv                                # Cleaned: Muni
│       ├── sfo_clean.csv                                 # Cleaned: SFO ground trips
│       ├── final_transportation_monthly.csv              # Final merged dataset
│       └── overview_plot.png                             # Time-series overview chart
└── 01-data-cleaning.ipynb                                # Data cleaning notebook
```

---

## Notebooks

### `01-data-cleaning.ipynb`
Loads, cleans, and merges all raw datasets into one analysis-ready monthly panel.

**What it does, step by step:**

| Section | Description |
|---|---|
| 1. Import packages | pandas, numpy, matplotlib, seaborn |
| 2. Load raw datasets | Loads all four sources with URL/API calls and local fallbacks; runs `.head()`, `.info()`, missing value checks |
| 3. Clean Waymo data | Parses dates, filters to 2023–2025, renames column to `waymo_passenger_miles` |
| 4. Clean taxi data | Fetches monthly aggregates via Socrata API (avoids loading 4M rows), computes `taxi_avg_fare` |
| 5. Clean Muni data | Pivots the 3-rows-per-month format into one wide row per month; extracts `muni_monthly_boardings` and `muni_recovery_rate` |
| 6. Clean SFO data | Aggregates airport ground trips by month; gracefully skips if API is unavailable |
| 7. Save intermediates | Saves `waymo_clean.csv`, `taxi_clean.csv`, `muni_clean.csv`, `sfo_clean.csv` |
| 8. Merge | Left join on `month` with Waymo as base; drops any duplicate columns as a safeguard |
| 9. Feature engineering | Creates expansion dummies, time trend, pct change, and log-transformed variables |
| 10. Final checks | Shape, summary stats, missing value table, overview time-series plot |
| 11. Save output | Saves `data/02-processed/final_transportation_monthly.csv` |

**Known bug fixed:** The raw Muni CSV stores three rows per month. When pivoting, the column name `"Baseline Monthly Total Boardings (accessible copy)"` contains `"Monthly Total Boardings"` as a substring. An earlier version of the rename logic matched both columns to the same name, creating a duplicate. Fixed by checking for `"Baseline"` before `"Monthly Total Boardings"` in the rename step.

---

## Raw Data Sources

All raw files are in the `data/` folder.

---

### 1. `passenger-miles-traveled-self-driving-taxis.csv`
**Source:** Our World in Data / California Public Utilities Commission (CPUC)
**Coverage:** March 2022 – December 2025 (monthly)
**Columns:**
- `Entity` — always "California" (statewide, not SF-specific)
- `Day` — last day of each month (YYYY-MM-DD)
- `Monthly distance traveled by robotaxi passengers` — total passenger miles for that month

**Notes:** California statewide data only — no publicly available SF-specific AV dataset exists. Used as the primary proxy for Waymo/robotaxi activity.

---

### 2. `sf-taxi.csv`
**Source:** DataSF — SF Open Data Portal (`data.sfgov.org`)
**Coverage:** December 2022 – October 2023 (~50,000 trip-level rows, sample)
**Columns:** `vehicle_placard_number`, `driver_id`, `start_time_local`, `end_time_local`, `pickup_location_latitude/longitude`, `dropoff_location_latitude/longitude`, `hail_type`, `paratransit`, `sfo_pickup`, `fare_type`, `meter_fare_amount`, `total_fare_amount`, `trip_distance_meters`, and others

**Notes:** This is a 50k-row sample — not used directly in the notebook. For monthly aggregate analysis, the notebook queries the full 4.1M-row dataset live via API. Full dataset: `https://data.sfgov.org/Transportation/Taxi-Trips/m8hk-2ipk`

---

### 3. `sf_taxi_monthly.csv`
**Source:** Derived from DataSF Taxi Trips dataset via Socrata API
**Coverage:** December 2022 – May 2024 (monthly aggregates)
**Columns:**
- `month` — year-month (YYYY-MM)
- `taxi_trips` — total number of SF taxi trips that month

**Notes:** Used as local fallback if the API is unavailable. No public SF taxi data exists past May 2024 — months after that are `NaN` in the final dataset.

---

### 4. `muni_ridership_monthly.csv`
**Source:** SFMTA Tableau Public dashboard (`transtat.sfmta.com`)
**URL:** `https://transtat.sfmta.com/t/public/views/SystemwideRidershipRecovery/MonthlySystemwideRecoveryAccessibleTable.csv`
**Coverage:** April 2020 – March 2026 (monthly)
**Raw format:** Long — three rows per calendar month, one per measure type:

| Measure Name | What it is |
|---|---|
| `Monthly Total Boardings (accessible copy)` | Total Muni boardings that month → becomes `muni_monthly_boardings` |
| `Baseline Monthly Total Boardings (accessible copy)` | 2019 pre-pandemic equivalent → becomes `muni_baseline_boardings` |
| `Monthly Recovery` | Ratio of current to baseline boardings → becomes `muni_recovery_rate` |

**Notes:** The notebook pivots this into one row per month. Excludes historic streetcar and cable car ridership (SFMTA APC methodology limitation).

---

### 5. `bart_monthly_exits.csv`
**Source:** DataSF — BART Daily Station Exits dataset via Socrata API (`data.sfgov.org/resource/m2xz-p7ja`)
**Coverage:** January 2022 – February 2025 (monthly aggregates)
**Columns:**
- `month` — year-month (YYYY-MM)
- `total_exits` — sum of all BART station exits system-wide that month

**Notes:** Bay Area-wide system totals, not SF stations only. Not included in the final merged dataset by default but available for supplementary analysis.

---

## Processed Data (`data/02-processed/`)

Generated by running `01-data-cleaning.ipynb` from top to bottom.

### `final_transportation_monthly.csv`
The main output. One row per month, January 2023 – December 2025 (36 rows).

| Column | Type | Description |
|---|---|---|
| `month` | string (YYYY-MM) | Calendar month |
| `waymo_passenger_miles` | float | CA statewide robotaxi passenger miles |
| `taxi_trips` | float | SF monthly taxi trip count (NaN after May 2024) |
| `taxi_total_fare` | float | SF monthly total taxi fares in dollars (NaN after May 2024) |
| `taxi_avg_fare` | float | Average fare per taxi trip that month (NaN after May 2024) |
| `muni_monthly_boardings` | float | Total Muni boardings that month |
| `muni_recovery_rate` | float | Muni ridership as a fraction of 2019 baseline |
| `sfo_ground_trips` | float | SFO airport ground transportation trips |
| `post_waymo_public_launch` | int (0/1) | 1 if month ≥ August 2023 |
| `post_waymo_open_to_all` | int (0/1) | 1 if month ≥ June 2024 |
| `time_trend` | int | Months since January 2023 (Jan 2023 = 0) |
| `waymo_pct_change` | float | Month-over-month % change in Waymo miles |
| `taxi_pct_change` | float | Month-over-month % change in taxi trips |
| `muni_pct_change` | float | Month-over-month % change in Muni boardings |
| `log_waymo_passenger_miles` | float | Natural log of Waymo passenger miles |
| `log_taxi_trips` | float | Natural log of taxi trips |
| `log_muni_monthly_boardings` | float | Natural log of Muni boardings |

### Intermediate cleaned files
- `waymo_clean.csv` — Waymo data filtered and renamed, one row per month
- `taxi_clean.csv` — Taxi data with fare variables, one row per month
- `muni_clean.csv` — Muni data pivoted to wide format, one row per month
- `sfo_clean.csv` — SFO ground trips aggregated by month

---

## Known Data Gaps & Limitations

| Gap | Reason | Workaround |
|---|---|---|
| No SF-specific Waymo/AV data | CPUC only reports statewide; no city-level public data exists | Use California statewide robotaxi miles as a proxy |
| No SF taxi data past May 2024 | DataSF dataset stopped updating after May 2024 | Acknowledge as limitation; months after May 2024 are NaN |
| No BART data past Feb 2025 | Source nulls in the dataset at download time | Use Jan 2022 – Feb 2025 window for supplementary analysis |
| Muni data excludes cable car & historic streetcar | SFMTA APC methodology limitation | Noted in SFMTA source documentation |

---

## Key Timeline Events (for Before/After Analysis)

- **August 2023** — Waymo opens commercial driverless service to the public in SF
- **June 2024** — Waymo One opens to everyone in SF (full public launch)
