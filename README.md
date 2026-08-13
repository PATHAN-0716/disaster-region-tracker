# Disaster Affected Region Tracker Analysis

**Category:** Data Engineering
**Skills Applied:** Python, MySQL, Data Visualization
**Project Type:** Geospatial Analytics / Disaster Management

---

## Table of Contents
- [Overview](#overview)
- [Objectives](#objectives)
- [Tech Stack](#tech-stack)
- [Repository Structure](#repository-structure)
- [Dataset Description](#dataset-description)
- [ETL Pipeline](#etl-pipeline)
- [Business Rules Implemented](#business-rules-implemented)
- [Database Design](#database-design)
- [Dashboards & Analytics](#dashboards--analytics)
- [How to Run](#how-to-run)
- [Sample Insights](#sample-insights)
- [Outcome](#outcome)
- [Future Improvements](#future-improvements)

---

## Overview

This project uses historical disaster records (floods, cyclones, earthquakes, droughts, landslides) to identify, monitor, and visualize disaster-affected regions across India. It simulates a real-world Data Engineering workflow: raw, messy CSV data is cleaned and standardized in Python, loaded into a properly constrained MySQL database, and then queried with SQL to power a set of Matplotlib dashboards. The end goal is to support disaster management authorities with actionable insights for risk assessment and emergency response planning.

## Objectives

- Analyze historical disaster data across multiple Indian states/regions
- Clean and standardize inconsistent, incomplete source data
- Design a relational database schema with proper constraints to store validated data
- Map and track disaster-affected regions by frequency, severity, and impact
- Provide visual, data-driven insights for emergency planning and policy support

## Tech Stack

| Layer            | Tools Used                                  |
|-------------------|----------------------------------------------|
| Data Wrangling     | Python, Pandas, NumPy                        |
| Database           | MySQL (via `mysql-connector-python`, SQLAlchemy) |
| Visualization      | Matplotlib                                   |
| Environment        | Google Colab (Jupyter Notebook)              |

## Repository Structure

```
disaster-region-tracker/
│
├── Disaster_Affected_Region_Tracker.ipynb   # Full ETL + MySQL + dashboard notebook
├── disaster_events.csv                       # Raw disaster event records
├── impact_assessment.csv                     # Raw impact data (people affected, losses)
├── regions.csv                                # Raw region reference data
├── requirements.txt                           # Python dependencies
└── README.md                                  # Project documentation (this file)
```

## Dataset Description

Three raw CSV files form the source data for this project:

**`disaster_events.csv`**
| Column          | Description                                      |
|------------------|---------------------------------------------------|
| `event_id`       | Unique ID for each disaster event                 |
| `disaster_type`  | Type of disaster (Flood, Cyclone, Earthquake, Drought, Landslide) |
| `region`         | Affected region/state name                        |
| `event_date`     | Date the disaster occurred                         |
| `severity`       | Severity level (Low, Medium, High)                 |

**`impact_assessment.csv`**
| Column                 | Description                              |
|-------------------------|--------------------------------------------|
| `impact_id`             | Unique ID for each impact record           |
| `event_id`              | Links to `disaster_events.event_id`        |
| `affected_people`       | Number of people affected                  |
| `economic_loss_musd`    | Economic loss in million USD               |

**`regions.csv`**
| Column         | Description                          |
|-----------------|----------------------------------------|
| `region_id`     | Unique ID for each region              |
| `region`        | Region/state name                      |
| `population`    | Population of the region               |
| `area_sq_km`    | Area of the region in sq. km           |

The raw data contains realistic data-quality issues: missing disaster types, missing/invalid dates, missing population values, missing affected-people and economic-loss figures, and duplicate rows/region entries — all of which are addressed in the ETL step below.

## ETL Pipeline

```
CSV files (dirty data)
        ↓
Python ETL (Pandas)
  - Deduplication
  - Null handling
  - Business rules
        ↓
MySQL clean tables (DDL + constraints)
        ↓
SQL analytics + Matplotlib dashboards
```

**Steps performed in the notebook:**
1. Load the three raw CSVs into Pandas DataFrames.
2. Drop exact duplicate rows in `disaster_events` and `impact_assessment`.
3. Drop duplicate region names in `regions` (keeping the first occurrence) to satisfy the `UNIQUE` constraint on region names in MySQL.
4. Apply the business rules below to handle nulls and invalid values.
5. Load the cleaned DataFrames into a MySQL database using SQLAlchemy, into tables created with explicit primary keys, foreign keys, and `NOT NULL` constraints.
6. Run SQL queries against MySQL to power five Matplotlib dashboard visuals.

## Business Rules Implemented

| Rule | Applied To | Logic |
|------|------------|-------|
| Replace missing disaster types with `"Unknown"` | `disaster_events.disaster_type` | `fillna('Unknown')` |
| Convert invalid dates safely | `disaster_events.event_date` | `pd.to_datetime(..., errors='coerce')` — invalid/unparseable dates become `NaT` instead of crashing the pipeline |
| Fill population with median | `regions.population` | `fillna(regions['population'].median())` |
| Missing affected people & losses → 0 | `impact_assessment.affected_people`, `impact_assessment.economic_loss_musd` | `fillna(0)` |
| Aggregate total affected people per region | Derived | `SUM(affected_people)` grouped by region via SQL join between `disaster_events` and `impact_assessment` |

## Database Design

Three related MySQL tables are created with referential integrity enforced through foreign keys:

```sql
regions (
    region_id INT PRIMARY KEY,
    region VARCHAR(100) NOT NULL UNIQUE,
    population BIGINT,
    area_sq_km BIGINT NOT NULL
)

disaster_events (
    event_id INT PRIMARY KEY,
    disaster_type VARCHAR(50) NOT NULL,
    region VARCHAR(100) NOT NULL,
    event_date DATE,
    severity VARCHAR(20) NOT NULL,
    FOREIGN KEY (region) REFERENCES regions(region)
)

impact_assessment (
    impact_id INT PRIMARY KEY,
    event_id INT NOT NULL,
    affected_people BIGINT NOT NULL DEFAULT 0,
    economic_loss_musd FLOAT NOT NULL DEFAULT 0,
    FOREIGN KEY (event_id) REFERENCES disaster_events(event_id)
)
```

**Entity relationship:** `regions` (1) → (many) `disaster_events` (1) → (many) `impact_assessment`

## Dashboards & Analytics

Each dashboard is generated in Matplotlib from the results of a live SQL query run against the cleaned MySQL tables:

1. **Top 5 regions by total affected population** — bar chart ranking regions by cumulative people affected, based on the aggregated SQL query joining `disaster_events` and `impact_assessment`.
2. **Disaster severity distribution by disaster type** — stacked bar chart showing counts of Low/Medium/High severity events per disaster type.
3. **Trend of disasters over time (monthly)** — line chart of disaster event counts aggregated by year-month.
4. **Economic loss vs affected population** — scatter plot exploring the relationship between people affected and economic loss (million USD) per event.
5. **Region-wise disaster frequency heatmap** — heatmap (region × disaster type) showing where specific disaster types cluster geographically.

## How to Run

1. Clone this repository:
   ```
   git clone https://github.com/PATHAN-0716/yourusername/disaster-region-tracker.git
   ```
2. Open `Disaster_Affected_Region_Tracker.ipynb` in Google Colab (or Jupyter with a local MySQL server).
3. Run the setup cell to install MySQL server and Python dependencies (Colab-specific; skip if MySQL is already running locally).
4. Run the upload cell and select `disaster_events.csv`, `impact_assessment.csv`, and `regions.csv` from this repo.
5. Run all remaining cells in order — ETL, database load, and dashboards will execute sequentially.

Alternatively, install dependencies locally with:
```
pip install -r requirements.txt
```

## Sample Insights

*(Fill in with your actual results after running the notebook, e.g.:)*
- Which region recorded the highest cumulative affected population
- Which disaster type is most frequently high-severity
- Whether disaster frequency is seasonal or increasing year over year
- Whether economic loss scales linearly with affected population, or if certain disaster types cause disproportionate loss

## Outcome

Maps and dashboards highlighting the most vulnerable regions by disaster frequency, severity, and human/economic impact — supporting policy planning and emergency response prioritization for disaster management authorities.

## Future Improvements

- Integrate real satellite/GIS layers for spatial mapping (e.g., via GeoPandas or Folium)
- Automate ingestion of real-time disaster feeds instead of static CSVs
- Add a live dashboard (Streamlit/Power BI) on top of the MySQL layer
- Expand severity classification using a quantitative impact score rather than categorical labels
