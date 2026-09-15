# Citi Bike Jersey City Data Analysis Project

This project analyzes Citi Bike trip data from Jersey City using Python and Tableau. It covers the full pipeline — from raw data acquisition through cleaning, weather enrichment, geospatial analysis, database storage, and an interactive dashboard.

## Links

- Dashboard: [[Tableau Public link](https://public.tableau.com/app/profile/vahe.yavryan/viz/CitiBikeJC2025/         JCCitiBikeDashboard)]
- Project Presentation: [[Presentation repo link](https://github.com/VahYavryan/presentation)]

## Overview

The pipeline pulls monthly Citi Bike trip data, daily weather data, and Jersey City neighborhood boundaries, cleans and merges them into an analysis-ready dataset, loads the result into a PostGIS-enabled PostgreSQL database, and visualizes it through a Tableau dashboard.

## Project Structure

```.
├── README.md
├── docker-compose.yaml
├── .env
├── .gitignore
├── data
│   ├── JC
│   ├── JC-202501-citibike-tripdata.csv
│   ├── JC-202502-citibike-tripdata.csv
│   ├── ...
│   └── JC-202512-citibike-tripdata.csv
├── notebook
│   ├── 01_download_citibike_data.ipynb
│   ├── 02_weather_data.ipynb
│   ├── 03_data_vizualization.ipynb
│   ├── 04_data_enrichment.ipynb
│   ├── 05_neighborhood_analysis.ipynb
│   └── 06_sqlalchemy_with_citibike.ipynb
├── postgis_data
└── tableau_dashboard
```

## Pipeline

### 1. Download Data from the Web

Automated Python scripts (`requests`, `pandas`, API clients) pull data from three sources:

| Source | Description | Format |

|---|---|---|
| **Citi Bike** | Monthly Jersey City trip files, 2025 (e.g. `202501.zip` … `202512.zip`) | ZIP → CSV |
| **Open-Meteo API** | Daily weather: max/min/mean temperature, precipitation, rain, snowfall, wind speed | JSON |
| **Jersey City Neighborhoods** | Neighborhood boundary polygons (Hudson County / Open Data) | GeoJSON |

**Sources:**

- Citi Bike: [s3.amazonaws.com/tripdata](https://s3.amazonaws.com/tripdata/index.html)
- Weather: [open-meteo.com](https://open-meteo.com)
- Neighborhoods: Hudson County Open Data portal

### 2. Raw Data Sources

- ~12 monthly Citi Bike ZIP files (CSV inside)
- Open-Meteo JSON response converted to a tabular DataFrame
- Neighborhood GeoJSON with polygon boundaries

### 3. Preparing the Data

| Step | What happens |

|---|---|
| **Extract** | Unzip and load CSV files |
| **Clean** | Rename columns, fix types, remove duplicates |
| **Parse** | Parse dates and times (`started_at`, `ended_at`) |
| **Validate** | Check and validate coordinates |
| **Handle Missing** | Handle missing values |
| **Standardize** | Standardize formats and units |

**Output:** cleaned DataFrames — `citibike_df`, `weather_df`, `neighborhoods_gdf`

### 4. Analysis with Python

| Analysis | Focus |

|---|---|
| **Exploratory Data Analysis (EDA)** | Distributions, summary stats, missing values |
| **Trend Analysis** | Daily / hourly / monthly trends, seasonality |
| **User & Bike Insights** | Member vs. casual, bike type share, usage patterns |
| **Spatial Analysis** | Top stations, neighborhood usage, spatial joins |
| **Weather Impact** | Temperature vs. rides, rain/snow impact, wind impact |
| **Feature Engineering** | Duration, distance, time features, weather flags |

**Output:** a dashboard-ready wide table combining all engineered features.

### 5. Create the Database (PostGIS / PostgreSQL, Docker)

Cleaned data is loaded into a PostGIS-enabled PostgreSQL database running in Docker.

- **Database:** `citibike`
- **Tables:**
  - `jersey_city` (rides)
  - `jersey_weather`
  - `jersey_city_neighborhoods` (geometry)
  - `jc_2025_stations` (geometry)
- **Materialized View:** `mv_tableau_citibike_dashboard` — the dashboard-ready dataset

### 6. Connect Tableau to the Database

| Setting | Value |
|---|---|
| Host | localhost |
| Port | 5432 |
| Database | citibike |
| Schema | public |
| Auth | Username / Password |

Tableau connects to the `mv_tableau_citibike_dashboard` materialized view as its data source.

**Benefits:** centralized data, fast queries, consistent logic, easy to maintain.

### 7. Build an Extract

For better dashboard performance, a Tableau `.hyper` extract is built from the materialized view (`citibike_dashboard.hyper`).

**Why extract instead of live connection?**

- Faster performance
- Optimized aggregation
- Works offline
- Smaller file size

### 8. Visualizing with Tableau

**Citi Bike Jersey City 2025 Dashboard** — an interactive, filterable dashboard covering:

- **KPIs:** Total Rides (999K), Avg Ride Duration (9,521 min), Avg Distance (1,221 km), Member Share (78%)
- **Trends:** Daily Rides, Rides by Hour of Day, Rides by Day of Week
- **Geography:** Top Start/End Neighborhoods, Top Start/End Stations (with maps)
- **Weather:** Rides vs. Temperature, Daily Rides vs. Temperature (dual-axis)
- **Detail Table:** Ride-level sample records
- **Global filters:** Date Range, Member Type, Bike Type

## Tech Stack

- **Python** — `pandas`, `requests`, `geopandas`, `sqlalchemy`, `geoalchemy2` (data acquisition, cleaning, enrichment, analysis, and database loading)
- **PostgreSQL + PostGIS** — spatial database storage (Docker container)
- **Tableau Desktop / Public** — interactive dashboard and visualization

## Notebooks

| Notebook | Purpose |
|---|---|
| `01_download_citibike_data.ipynb` | Download monthly Citi Bike trip data |
| `02_weather_data.ipynb` | Pull daily weather data from the Open-Meteo API |
| `03_data_vizualization.ipynb` | Exploratory data analysis and Python-based visualizations |
| `04_data_enrichment.ipynb` | Clean, validate, and enrich the raw trip data |
| `05_neighborhood_analysis.ipynb` | Geospatial joins and neighborhood-level analysis |
| `06_sqlalchemy_with_citibike.ipynb` | Load the cleaned data into PostGIS/PostgreSQL via SQLAlchemy and build the `mv_tableau_citibike_dashboard` materialized view |

## How to Run

1. Clone the repository and set up a Python environment with the required packages (`pandas`, `geopandas`, `sqlalchemy`, `geoalchemy2`, etc.).
2. Copy `.env.example` to `.env` (or create `.env`) and fill in your database credentials.
3. Run the notebooks in order (`01` → `06`) to download, clean, enrich, analyze, and load the data:
   - `01_download_citibike_data.ipynb`
   - `02_weather_data.ipynb`
   - `03_data_vizualization.ipynb`
   - `04_data_enrichment.ipynb`
   - `05_neighborhood_analysis.ipynb`
   - `06_sqlalchemy_with_citibike.ipynb` — loads data into PostGIS and creates the materialized view
4. Start the PostGIS/PostgreSQL database with `docker-compose up -d` (uses `docker-compose.yaml`; data persists in `postgis_data`).
5. Connect Tableau Desktop to the `mv_tableau_citibike_dashboard` materialized view.
6. Build a `.hyper` extract for optimal dashboard performance.
7. Open the Tableau dashboard (see `tableau_dashboard`) to explore ridership, station, and weather insights.

## Data Sources

- Citi Bike System Data: [s3.amazonaws.com/tripdata](https://s3.amazonaws.com/tripdata/index.html)
- Open-Meteo Weather API: [open-meteo.com](https://open-meteo.com)
- Jersey City Neighborhood Boundaries: Hudson County Open Data
