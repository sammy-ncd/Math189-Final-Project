# Math 189 Final Project

**Research Question:** For San Francisco from 2023 to 2025, did the expansion of Waymo's driverless ride-hailing service correspond with changes in taxi activity, public transit ridership, and the broader transportation market?

---

## Data Sources

All raw data files are in the `data/` folder.

---

### 1. `passenger-miles-traveled-self-driving-taxis.csv`
**Source:** Our World in Data / California Public Utilities Commission (CPUC)
**Coverage:** March 2022 – December 2025 (monthly)
**Columns:**
- `Entity` — always "California" (statewide, not SF-specific)
- `Day` — last day of each month (YYYY-MM-DD)
- `Monthly distance traveled by robotaxi passengers` — total passenger miles for that month

**Notes:** This is California statewide data, not San Francisco only. No publicly available SF-specific AV dataset exists. Used as the primary proxy for Waymo/robotaxi activity.

---

### 2. `sf-taxi.csv`
**Source:** DataSF — SF Open Data Portal (`data.sfgov.org`)
**Coverage:** December 2022 – October 2023 (~50,000 trip-level rows, sample)
**Columns:** `vehicle_placard_number`, `driver_id`, `start_time_local`, `end_time_local`, `pickup_location_latitude/longitude`, `dropoff_location_latitude/longitude`, `hail_type`, `paratransit`, `sfo_pickup`, `fare_type`, `meter_fare_amount`, `total_fare_amount`, `trip_distance_meters`, and others

**Notes:** This is a 50k-row sample of the full DataSF Taxi Trips dataset (4.1 million rows, Dec 2022 – May 2024). For monthly aggregate analysis, use `sf_taxi_monthly.csv` instead. Full dataset at: `https://data.sfgov.org/Transportation/Taxi-Trips/m8hk-2ipk`

---

### 3. `sf_taxi_monthly.csv`
**Source:** Derived from DataSF Taxi Trips dataset via Socrata API (`data.sfgov.org/resource/m8hk-2ipk`)
**Coverage:** December 2022 – May 2024 (monthly aggregates)
**Columns:**
- `month` — year-month (YYYY-MM)
- `taxi_trips` — total number of SF taxi trips that month

**Notes:** Aggregated from the full 4.1M-row DataSF dataset. No public SF taxi data exists past May 2024 — this is a known limitation. Data after May 2024 is not available through any public source.

---

### 4. `muni_ridership_monthly.csv`
**Source:** SFMTA Tableau Public dashboard (`transtat.sfmta.com`)
**URL:** `https://transtat.sfmta.com/t/public/views/SystemwideRidershipRecovery/MonthlySystemwideRecoveryAccessibleTable.csv`
**Coverage:** April 2020 – March 2026 (monthly)
**Columns:**
- `Measure Names` — one of three metrics per row: `Monthly Recovery`, `Monthly Total Boardings (accessible copy)`, or `Baseline Monthly Total Boardings (accessible copy)`
- `Month of MONTH` — month label (e.g., "January 2023")
- `Measure Values` — numeric value for that metric

**Notes:** Each calendar month has three rows (one per measure). "Monthly Total Boardings" is the key variable for this project. "Baseline" is the 2019 pre-pandemic equivalent. "Monthly Recovery" is the ratio of current to baseline boardings. Excludes historic streetcar and cable car ridership.

---

### 5. `bart_monthly_exits.csv`
**Source:** DataSF — BART Daily Station Exits dataset via Socrata API (`data.sfgov.org/resource/m2xz-p7ja`)
**Coverage:** January 2022 – February 2025 (monthly aggregates)
**Columns:**
- `month` — year-month (YYYY-MM)
- `total_exits` — sum of all BART station exits system-wide that month

**Notes:** Aggregated from daily station-level data. Covers the full BART system (Bay Area-wide), not SF stations only. Data past February 2025 had null totals in the source dataset at time of download. BART data is produced by BART, not the City of SF.

---

## Known Data Gaps & Limitations

| Gap | Reason | Workaround |
|---|---|---|
| No SF-specific Waymo/AV data | CPUC only reports statewide; no city-level public data exists | Use California statewide robotaxi miles as a proxy |
| No SF taxi data past May 2024 | DataSF dataset stopped updating after May 2024 | Acknowledge as limitation; use Dec 2022 – May 2024 window |
| No BART data past Feb 2025 | Source nulls in the dataset at download time | Use Jan 2022 – Feb 2025 window |
| Muni data excludes cable car & historic streetcar | SFMTA APC methodology limitation | Noted in SFMTA source documentation |

---

## Key Timeline Events (for Before/After Analysis)

- **August 2023** — Waymo opens commercial driverless service to the public in SF
- **June 2024** — Waymo One opens to everyone in SF (full public launch)
