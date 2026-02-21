# Energy Consumption ETL Pipeline

A data engineering project that processes household electricity consumption data using Databricks.

## What This Project Does

This project takes raw electricity usage data from a smart meter system and processes it through three stages:
1. **Bronze**: Store raw data as-is
2. **Silver**: Clean and organize the data
3. **Gold**: Create summary tables for analysis

The goal is to help an energy analytics team understand when and how electricity is being used in a household.

## The Data

**Source**: UCI Machine Learning Repository - Individual Household Electric Power Consumption  
**Size**: Over 2 million rows of minute-by-minute data  
**Download**: https://archive.ics.uci.edu/ml/machine-learning-databases/00235/household_power_consumption.zip

The dataset tracks:
- Date and time of measurements
- Total power being used (global active power)
- Voltage and current levels
- Three sub-meters tracking different areas of the house:
  - Sub-meter 1: Kitchen appliances
  - Sub-meter 2: Laundry room
  - Sub-meter 3: Water heater and air conditioner

## How the Pipeline Works

### Architecture Diagram

![Pipeline Architecture](docs/internship_architecture.drawio.svg)

The diagram shows how data flows from the raw source through three layers: Bronze (raw), Silver (cleaned), and Gold (summary tables).

### Bronze Layer

**File**: `notebooks/01_bronze.ipynb`

This layer downloads the raw data file and stores it without changes.

What it does:
- Downloads ZIP file from the UCI website
- Extracts the text file
- Reads it with semicolon separators
- Adds two metadata columns:
  - `_ingestion_timestamp`: When the data was loaded
  - `_source_file_name`: Where the file came from
- Saves as a Delta table

Output table: `energy_demand_catalog.{env}.raw_power_usage`

### Silver Layer

**File**: `notebooks/02_silver.ipynb`

This layer cleans and prepares the data for analysis.

Cleaning steps:
- Combines Date and Time columns into one datetime column
- Converts numbers from text to decimal format
- Replaces missing values (shown as "?") with empty values
- Removes rows with bad timestamps
- Filters out negative power readings
- Renames all columns to use lowercase with underscores

**Sub-meter Unit Conversion:**
- Sub-meter values were converted from Wh to kWh (divided by 1000)
- This was necessary because Power BI calculations revealed the original readings were in Wh, not kWh
- Sub-meter readings represent only a portion of household consumption (some appliances are unmetered)
- Unmetered consumption is calculated in Power BI using DAX
- The original column names were preserved to avoid schema mismatch with the expected project structure

New columns added:
- `hour_of_day`: Hour number (0-23)
- `day_of_week`: Day name (Monday, Tuesday, etc.)
- `is_weekend`: True if Saturday or Sunday
- `consumption_kwh`: Energy used in kilowatt-hours

Output table: `energy_demand_catalog.{env}.silver_power_consumption`

Also exports cleaned data as a CSV file.

### Gold Layer

**File**: `notebooks/03_gold.ipynb`

This layer creates three summary tables for business analysis.

#### Table 1: Hourly Summary
**Name**: `agg_hourly_metrics`

Shows average usage patterns by hour of day:
- hour_of_day
- avg_consumption_kwh
- min_voltage
- max_voltage
- avg_current_intensity

Purpose: Find which hours of the day use the most electricity.

#### Table 2: Daily Summary
**Name**: `agg_daily_metrics`

Shows daily totals and patterns:
- date
- total_kwh (total energy used that day)
- avg_kwh (average per measurement)
- peak_hour (which hour had the highest usage)
- weekend_flag (1 if weekend, 0 if weekday)

Purpose: Compare weekdays to weekends and track daily trends.

#### Table 3: Sub-Meter Summary
**Name**: `agg_submeter_metrics`

Shows daily totals by area of the house:
- date
- sub_meter_1_total (kitchen)
- sub_meter_2_total (laundry)
- sub_meter_3_total (water heater and AC)

Purpose: See which parts of the house use the most energy.

## Project Files

```
energy-demand-etl/
├── config/
│   └── setup.ipynb              # Setup code
├── notebooks/
│   ├── 01_bronze.ipynb          # Bronze layer
│   ├── 02_silver.ipynb          # Silver layer  
│   └── 03_gold.ipynb            # Gold layer
├── docs/
│   └── internship_architecture.drawio.svg  # Diagram
└── README.md                    # This file
```

## How to Run

### Step 1: Setup
Run `config/setup.ipynb` first. This creates:
- A catalog called `energy_demand_catalog`
- A volume for storing files
- Table name variables

Choose environment (dev or prod) using the dropdown widget at the top of the notebook.

### Step 2: Bronze Layer
Run `notebooks/01_bronze.ipynb`

This downloads and stores the raw data.

### Step 3: Silver Layer  
Run `notebooks/02_silver.ipynb`

This cleans the data and creates the silver table.

### Step 4: Gold Layer
Run `notebooks/03_gold.ipynb`

This creates the three summary tables and shows sample data from each.

## What You Get

After running all notebooks, you will have:

1. **Four Delta tables** in Databricks:
   - raw_power_usage (bronze)
   - silver_power_consumption (silver)
   - agg_hourly_metrics (gold)
   - agg_daily_metrics (gold)
   - agg_submeter_metrics (gold)

2. **Cleaned data CSV** exported to your volume storage

3. **Sample outputs** displayed in the Gold notebook

## Key Findings

The Gold tables help answer questions like:

- What time of day uses the most electricity?
- Do weekends use more or less power than weekdays?
- Which uses more energy: the kitchen, laundry, or water heater?
- What was the highest usage day?

Example insights:
- Evening hours (6 PM to 9 PM) usually have peak usage
- Water heater and AC (sub-meter 3) typically account for the most consumption
- Weekends may show different patterns than weekdays

## Technical Details

**Platform**: Databricks  
**Language**: Python with PySpark  
**Storage**: Delta Lake tables  
**Architecture**: Medallion (Bronze-Silver-Gold)

## Requirements Met

This project fulfills all internship requirements:

- Bronze layer with raw data and metadata columns
- Silver layer with data cleaning and new features
- Gold layer with three analytical tables
- All tables include display() for viewing results
- Architecture diagram included
- Cleaned data exported as CSV
- Well-commented code throughout

## Future Extensions

This project can be extended later with:

- Data visualization dashboards
- Demand forecasting models

## Notes

- The pipeline runs in batch mode (not streaming)
- Environment can be toggled between dev and prod
- Data is downloaded once and reused in dev mode
- All column names use snake_case format

---

Created for Data Engineering Internship
