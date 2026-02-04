# Data Dictionary – Rush Hour Reality

This folder contains the raw GTFS‑Realtime snapshots, GTFS Static reference files, and the processed dataset used to power the Rush Hour Reality visualisation.

---

## Dataset Overview

- **Source:** National Transport Authority / Transport for Ireland Open Data (GTFS‑Realtime + GTFS Static).  
- **Geographic focus:** Greater Dublin Area (filtered by latitude 53.15–53.60, longitude −6.60 to −6.00 in the processing pipeline).  
- **Time period:** 13 consecutive days of collection (November 2025), with snapshots every 30 minutes.  
- **Record volume:**  
  - Trip updates and vehicle positions combined: ~1.5M raw records across all snapshots.  
  - Processed dataset: one record per bus–stop–snapshot with derived delay metrics.  

Data is grouped into three layers:

1. **Raw GTFS‑Realtime snapshots** (`trip_updates_*.csv`, `vehicle_positions_*.csv`)  
2. **GTFS Static reference** (`routes.txt`, `trips.txt`, `stops.txt`)  
3. **Processed analysis table** (`processed_data.csv`) used for aggregation and visualisation.  

---

## 1️⃣ Raw GTFS‑Realtime – Trip Updates (`trip_updates_YYYYMMDD_HHMMSS.csv`)

Each file is a parsed snapshot of the GTFS‑Realtime **TripUpdates** feed at a specific timestamp. [file:37]

| Field Name            | Type      | Description                                               | Example                          |
|-----------------------|-----------|-----------------------------------------------------------|----------------------------------|
| `collection_timestamp`| string    | Time the snapshot was collected (ISO‑8601).              | `2025-11-10T03:00:01+00:00`      |
| `entity_id`           | string    | Unique identifier of the GTFS entity in the feed.        | `T1`                             |
| `trip_id`             | string    | Trip identifier from GTFS Static `trips.txt`.            | `5058_31784`                     |
| `route_id`            | string    | Route identifier from `routes.txt`.                      | `5058_112921`                    |
| `direction_id`        | int       | Trip direction (0 = outbound, 1 = inbound).              | `0`                              |
| `start_date`          | int       | Service date in `YYYYMMDD` format.                       | `20251109`                       |
| `start_time`          | string    | Scheduled start time of the trip (can exceed 24h clock). | `23:00:00`                       |
| `vehicle_id`          | float/int | Vehicle identifier (may be missing).                     | `1321` or empty                  |
| `vehicle_label`       | string    | Human readable vehicle label (often empty).              | *(blank)*                        |
| `stop_sequence`       | int       | Position of the stop within the trip.                    | `17`                             |
| `stop_id`             | string    | Stop identifier referencing `stops.txt`.                 | `8490B141631`                    |
| `arrival_delay`       | float     | Arrival delay in seconds relative to schedule.           | `638.0`                          |
| `arrival_time`        | string    | Updated arrival time if provided.                        | *(blank in sample)*              |
| `departure_delay`     | float     | Departure delay in seconds.                              | `638.0`                          |
| `departure_time`      | string    | Updated departure time if provided.                      | *(blank in sample)*              |

---

## 2️⃣ Raw GTFS‑Realtime – Vehicle Positions (`vehicle_positions_YYYYMMDD_HHMMSS.csv`)

Each file is a parsed snapshot of the **VehiclePositions** feed at a specific timestamp. [file:38]

| Field Name              | Type      | Description                                                       | Example                          |
|-------------------------|-----------|-------------------------------------------------------------------|----------------------------------|
| `collection_timestamp`  | string    | Time the snapshot was collected (ISO‑8601).                      | `2025-11-10T03:00:04+00:00`      |
| `entity_id`             | string    | Unique identifier of the vehicle entity in the feed.             | `V1`                             |
| `vehicle_id`            | int       | Vehicle identifier.                                               | `1321`                           |
| `vehicle_label`         | string    | Human‑readable label (often empty).                              | *(blank)*                        |
| `trip_id`               | string    | Linked trip identifier (`trips.txt`).                            | `5058_32834`                     |
| `route_id`              | string    | Linked route identifier (`routes.txt`).                          | `5058_112973`                    |
| `direction_id`          | int       | Direction of travel (0 or 1).                                    | `0`                              |
| `start_date`            | int       | Service date in `YYYYMMDD`.                                      | `20251109`                       |
| `start_time`            | string    | Trip start time (may exceed 24h clock for late‑night services).  | `26:30:00`                       |
| `latitude`              | float     | Vehicle latitude.                                                 | `51.89664459228516`              |
| `longitude`             | float     | Vehicle longitude.                                                | `-8.474621772766113`             |
| `bearing`               | float     | Compass bearing (degrees).                                       | *(blank)*                        |
| `speed`                 | float     | Vehicle speed (m/s, may be missing).                             | *(blank)*                        |
| `current_stop_sequence` | int       | Index of the next stop.                                          | `0`                              |
| `stop_id`               | string    | Next stop identifier (often empty in sample).                    | *(blank)*                        |
| `current_status`        | int       | Status code (e.g. 0 = in transit).                               | `0`                              |
| `timestamp`             | float     | Vehicle position timestamp (epoch seconds).                      | `1.762743594e+09`                |
| `congestion_level`      | int       | Coded congestion level from GTFS (0 = unknown / none).           | `0`                              |

---

## 3️⃣ GTFS Static – Routes (`routes.txt`)

Reference table describing each route. [file:42]

| Field Name         | Type   | Description                                  | Example                                                                 |
|--------------------|--------|----------------------------------------------|-------------------------------------------------------------------------|
| `route_id`         | string | Primary key linking to `trips.txt`.          | `3113_32198`                                                            |
| `agency_id`        | int    | Operating agency identifier.                 | `7778347`                                                               |
| `route_short_name` | string | Short public name or code of the route.      | `DK05`                                                                  |
| `route_long_name`  | string | Descriptive route name.                      | `Cabra (Cavan), Forest Park - Dundalk I.T (Main Gate)`                 |
| `route_desc`       | string | Additional description (often blank).        | *(blank)*                                                               |
| `route_type`       | int    | Transport mode (3 = bus).                    | `3`                                                                     |
| `route_url`        | string | Web page for the route (if provided).        | *(blank)*                                                               |
| `route_color`      | string | Hex colour used for route branding.          | *(blank in sample)*                                                     |
| `route_text_color` | string | Hex colour for text on route signage.        | *(blank in sample)*                                                     |

---

## 4️⃣ GTFS Static – Trips (`trips.txt`)

Trip‑level metadata used to interpret GTFS‑Realtime events. [file:41]

| Field Name       | Type   | Description                                             | Example                                |
|------------------|--------|---------------------------------------------------------|----------------------------------------|
| `route_id`       | string | Foreign key to `routes.route_id`.                      | `3113_32198`                           |
| `service_id`     | int    | Service pattern identifier (weekday, weekend, etc.).   | `1`                                    |
| `trip_id`        | string | Unique trip identifier (joins to realtime feeds).      | `3113_1`                               |
| `trip_headsign`  | string | Destination / headsign text shown to passengers.       | `Dundalk IT, stop 138951`             |
| `trip_short_name`| string | Short internal name for the trip.                      | `1.Mo-Fr.43-DK0-5-y11`                |
| `direction_id`   | int    | Direction of travel (0 or 1).                          | `0`                                    |
| `block_id`       | string | Block/group of trips operated by the same vehicle.     | `3113_7778350_Txc123802`              |
| `shape_id`       | string | Identifier linking to a shape (polyline of the route). | `3113_1`                               |

---

## 5️⃣ GTFS Static – Stops (`stops.txt`)

Stop‑level metadata including names and coordinates. [file:40]

| Field Name       | Type    | Description                                     | Example                      |
|------------------|---------|-------------------------------------------------|------------------------------|
| `stop_id`        | string  | Unique stop identifier.                         | `701000002`                  |
| `stop_code`      | int     | Public stop code.                               | `12537`                      |
| `stop_name`      | string  | Name of the stop.                               | `Derry Patrick Street`       |
| `stop_desc`      | string  | Additional description (often blank).          | *(blank)*                    |
| `stop_lat`       | float   | Latitude of the stop.                           | `54.99998`                   |
| `stop_lon`       | float   | Longitude of the stop.                          | `-7.32234`                   |
| `zone_id`        | string  | Fare zone (if provided).                        | *(blank)*                    |
| `stop_url`       | string  | Web link for stop information.                  | *(blank)*                    |
| `location_type`  | int     | 0 = stop, 1 = station, etc. (often blank).     | *(blank)*                    |
| `parent_station` | string  | Parent station for complex interchanges.        | *(blank)*                    |

---

## 6️⃣ Processed Dataset – Analysis Table (`processed_data.csv`)

This is the main table used for analysis and visualisation after joining realtime data with static references and engineering features. [file:39]

| Field Name          | Type     | Description                                                              | Example                                          |
|---------------------|----------|--------------------------------------------------------------------------|--------------------------------------------------|
| `collection_timestamp` | string | Snapshot time from the original realtime feed.                           | `2025-11-07T23:21:10+00:00`                      |
| `entity_id`         | string   | GTFS entity identifier from TripUpdates.                                 | `T4`                                             |
| `trip_id`           | string   | GTFS trip identifier.                                                    | `5058_48201`                                     |
| `route_id`          | string   | GTFS realtime route identifier.                                          | `5058_112924`                                    |
| `direction_id`      | int      | Trip direction (0/1).                                                    | `1`                                              |
| `start_date`        | int      | Service date (`YYYYMMDD`).                                              | `20251107`                                       |
| `start_time`        | string   | Trip start time.                                                         | `18:30:00`                                       |
| `vehicle_id`        | int      | Vehicle identifier.                                                      | `7128`                                           |
| `vehicle_label`     | string   | Vehicle label (if available).                                            | *(blank)*                                        |
| `stop_sequence`     | int      | Stop index within the trip.                                             | `16`                                             |
| `stop_id`           | string   | Stop identifier.                                                         | `8240B111911`                                    |
| `arrival_delay`     | float    | Arrival delay in seconds.                                               | `5978`                                           |
| `arrival_time`      | string   | Updated arrival time (if present).                                      | *(blank)*                                        |
| `departure_delay`   | float    | Departure delay in seconds (may be empty).                              | *(blank)*                                        |
| `departure_time`    | string   | Updated departure time (if present).                                    | *(blank)*                                        |
| `route_id_ref`      | string   | Route id from static GTFS, used for joins.                              | `5058_112924`                                    |
| `route_short_name`  | string   | Public route code.                                                       | `32`                                             |
| `route_long_name`   | string   | Descriptive route name.                                                 | `Dublin Airport - Monaghan - Letterkenny`        |
| `stop_name`         | string   | Stop name from `stops.txt`.                                             | `Dublin Airport Terminal 2 Zone 19`              |
| `stop_lat`          | float    | Stop latitude.                                                           | `53.42611`                                       |
| `stop_lon`          | float    | Stop longitude.                                                          | `-6.2384`                                        |
| `datetime`          | string   | Combined timestamp (`collection_timestamp` normalised).                 | `2025-11-07T23:21:10.000000+0000`                |
| `delay_minutes`     | float    | Arrival delay converted into minutes.                                   | `99.63333333333333`                              |
| `hour`              | int      | Hour of day (0–23) derived from `datetime`.                             | `23`                                             |
| `date`              | string   | Calendar date (`YYYY-MM-DD`).                                           | `2025-11-07`                                     |
| `weekday`           | int      | Day of week (0 = Monday … 6 = Sunday).                                  | `5`                                              |
| `time_period`       | string   | Categorical time band (e.g. Peak, Off Peak).                            | `Off Peak`                                       |
| `delay_category`    | string   | Binned delay label (e.g. On Time, Minor Delay, Major Delay).            | `Major Delay`                                    |

---

## Data Quality Notes

- **Missing values:** Some realtime fields (vehicle label, speed, bearing, updated times) are frequently empty; analysis focuses on delay seconds and positions that are reliably populated.  
- **Outliers:** Extremely large delays are preserved but may be filtered or capped during analysis when building charts.  
- **Coverage gaps:** A few snapshots are missing when the collection machine was offline; the 24‑hour aggregate model smooths these gaps across 13 days.  

---

## Data Collection Methodology (High Level)

1. Poll GTFS‑Realtime **TripUpdates** and **VehiclePositions** endpoints every 30 minutes using a Python script scheduled via Windows Task Scheduler.  
2. Persist each snapshot to timestamped CSV files inside the `data/raw/` folder.  
3. Join realtime records with GTFS Static routes/trips/stops to enrich with human‑readable names and coordinates.  
4. Filter to the Greater Dublin Area and engineer features (`delay_minutes`, `hour`, `time_period`, `delay_category`) before exporting `processed_data.csv` for analysis and visualisation.  

---

This `data/README.md` should live alongside your CSV/TXT files under `data/`, so anyone cloning the repo can quickly understand what each file contains and how it was constructed.
