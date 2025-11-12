# 🧹 Module 2: Data Cleaning and Feature Engineering

## 📋 Table of Contents
- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Data Cleaning Pipeline](#data-cleaning-pipeline)
- [Feature Engineering](#feature-engineering)
- [Usage](#usage)
- [Input Data Format](#input-data-format)
- [Output Data Format](#output-data-format)
- [Configuration](#configuration)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)

---

## 🎯 Overview

This module provides a comprehensive data preprocessing pipeline for environmental datasets, transforming raw sensor data into clean, feature-rich datasets ready for machine learning applications. It handles data quality issues, standardizes formats, and creates meaningful features to improve model performance.

**Key Capabilities:**
- ✅ Data quality validation and cleaning
- 🔄 Missing value imputation
- 📏 Feature scaling and normalization
- 🗺️ Spatial feature engineering
- ⏰ Temporal feature extraction
- 🔗 Multi-source data integration

---

## 🔧 Prerequisites

### Required Dependencies
```bash
python >= 3.8
pandas >= 1.3.0
numpy >= 1.21.0
scikit-learn >= 1.0.0
scipy >= 1.7.0
geopy >= 2.2.0
```

### Optional Dependencies (for advanced features)
```bash
geopandas >= 0.10.0  # For spatial operations
shapely >= 1.8.0     # For geometric calculations
```

---

## 📦 Installation

```bash
# Clone the repository
git clone <repository-url>
cd module-2-data-cleaning

# Install dependencies
pip install -r requirements.txt

# Or using conda
conda env create -f environment.yml
conda activate env-data-cleaning
```

---

## 🧼 Data Cleaning Pipeline

### 1. Remove Duplicates and Invalid Records

**Objective:** Ensure data integrity by removing redundant and erroneous entries.

```python
from data_cleaning import remove_duplicates, validate_records

# Remove exact duplicates
df_cleaned = remove_duplicates(df, subset=['timestamp', 'location_id'])

# Filter invalid sensor readings
df_valid = validate_records(
    df_cleaned,
    rules={
        'PM2.5': (0, 500),      # μg/m³
        'PM10': (0, 1000),      # μg/m³
        'NO2': (0, 200),        # μg/m³
        'SO2': (0, 100),        # μg/m³
        'CO': (0, 50),          # ppm
        'O3': (0, 300),         # μg/m³
        'temperature': (-50, 60),  # °C
        'humidity': (0, 100)    # %
    }
)
```

**Features:**
- Exact and near-duplicate detection
- Range-based validation for sensor readings
- Configurable thresholds per pollutant
- Logging of removed records for audit

### 2. Handle Missing Values

**Objective:** Impute missing data intelligently without introducing bias.

```python
from data_cleaning import handle_missing_values

df_imputed = handle_missing_values(
    df_valid,
    methods={
        'PM2.5': 'interpolate',      # Time-series interpolation
        'temperature': 'forward_fill', # Forward fill
        'humidity': 'median',         # Median imputation
        'wind_speed': 'mean'          # Mean imputation
    },
    max_gap=6  # Maximum consecutive missing values to interpolate
)
```

**Supported Methods:**
- **Interpolation:** Linear, polynomial, spline for time-series data
- **Statistical:** Mean, median, mode imputation
- **Forward/Backward Fill:** For temporally correlated data
- **KNN Imputation:** For complex patterns
- **Deletion:** For excessive missing data (>30%)

### 3. Standardize Data Formats

**Objective:** Ensure consistency across datasets and sources.

```python
from data_cleaning import standardize_formats

df_standardized = standardize_formats(
    df_imputed,
    config={
        'timestamp': 'ISO8601',           # YYYY-MM-DD HH:MM:SS
        'coordinates': 'decimal_degrees',  # (lat, lon)
        'units': {
            'PM2.5': 'μg/m³',
            'PM10': 'μg/m³',
            'NO2': 'μg/m³',
            'temperature': '°C',
            'pressure': 'hPa'
        }
    }
)
```

**Standardization Tasks:**
- Convert timestamps to UTC with timezone awareness
- Standardize GPS coordinates to WGS84 format
- Unify pollutant units (μg/m³, ppm, ppb)
- Normalize categorical labels

### 4. Normalize Feature Values

**Objective:** Scale features for optimal model performance.

```python
from data_cleaning import normalize_features

df_normalized, scalers = normalize_features(
    df_standardized,
    method='standard',  # Options: 'standard', 'minmax', 'robust'
    features=['PM2.5', 'PM10', 'NO2', 'SO2', 'CO', 'O3', 
              'temperature', 'humidity', 'wind_speed']
)

# Save scalers for inverse transformation
import joblib
joblib.dump(scalers, 'models/feature_scalers.pkl')
```

**Scaling Methods:**
- **Standard Scaler:** Zero mean, unit variance (Gaussian distribution)
- **MinMax Scaler:** Scale to [0, 1] range
- **Robust Scaler:** Median and IQR-based (outlier-resistant)

---

## 🚀 Feature Engineering

### 5. Generate Spatial Features

**Objective:** Create location-based features capturing environmental context.

```python
from feature_engineering import generate_spatial_features

df_spatial = generate_spatial_features(
    df_normalized,
    reference_data={
        'roads': 'data/spatial/road_network.geojson',
        'industrial': 'data/spatial/industrial_zones.geojson',
        'dump_sites': 'data/spatial/waste_facilities.geojson',
        'water_bodies': 'data/spatial/rivers_lakes.geojson'
    },
    features=[
        'distance_to_nearest_road',
        'distance_to_industrial',
        'distance_to_dump_site',
        'road_density_500m',      # Roads within 500m radius
        'elevation',
        'land_use_type'
    ]
)
```

**Spatial Features:**
- Distance to nearest infrastructure (roads, industries, airports)
- Proximity to pollution sources (factories, landfills, traffic)
- Road/building density in buffer zones
- Elevation and topographical features
- Land use classification

### 6. Derive Temporal Features

**Objective:** Extract time-based patterns to capture pollution dynamics.

```python
from feature_engineering import derive_temporal_features

df_temporal = derive_temporal_features(
    df_spatial,
    timestamp_col='timestamp',
    features=[
        'hour',              # 0-23
        'day_of_week',       # 0-6 (Monday=0)
        'month',             # 1-12
        'season',            # winter/spring/summer/fall
        'is_weekend',        # Boolean
        'is_rush_hour',      # Peak traffic times
        'day_of_year',       # 1-365
        'week_of_year'       # 1-52
    ],
    cyclical_encoding=True  # Encode hour/month as sine/cosine
)
```

**Temporal Features:**
- Hour of day (captures diurnal patterns)
- Day of week (weekday vs. weekend differences)
- Month/Season (captures seasonal variations)
- Holiday indicators (special event patterns)
- Cyclical encoding for periodic features

### 7. Integrate All Data Sources

**Objective:** Combine multiple datasets into a unified feature matrix.

```python
from feature_engineering import integrate_datasets

df_final = integrate_datasets(
    datasets={
        'pollution': df_temporal,
        'weather': 'data/processed/weather_data.csv',
        'traffic': 'data/processed/traffic_counts.csv',
        'satellite': 'data/processed/satellite_aod.csv'
    },
    merge_keys={
        'pollution': ['timestamp', 'location_id'],
        'weather': ['timestamp', 'station_id'],
        'traffic': ['timestamp', 'road_segment_id'],
        'satellite': ['date', 'grid_cell_id']
    },
    merge_strategy='left',  # Keep all pollution records
    time_tolerance='1H'     # Match within 1 hour
)

# Save final dataset
df_final.to_csv('data/processed/final_feature_dataset.csv', index=False)
```

**Integration Features:**
- Multi-source data merging
- Temporal and spatial alignment
- Handling of different sampling frequencies
- Feature completeness validation

---

## 💻 Usage

### Basic Pipeline Execution

```python
from pipeline import DataCleaningPipeline

# Initialize pipeline
pipeline = DataCleaningPipeline(config='config/pipeline_config.yaml')

# Load raw data
raw_data = pipeline.load_data('data/raw/sensor_readings.csv')

# Execute full pipeline
cleaned_data = pipeline.run(
    data=raw_data,
    steps=[
        'remove_duplicates',
        'validate_records',
        'handle_missing',
        'standardize_formats',
        'normalize_features',
        'generate_spatial',
        'derive_temporal',
        'integrate_sources'
    ]
)

# Generate cleaning report
pipeline.generate_report('reports/cleaning_report.html')
```

### Advanced: Custom Pipeline

```python
# Create custom pipeline
from pipeline import Pipeline

custom_pipeline = Pipeline()
custom_pipeline.add_step('remove_duplicates', subset=['timestamp', 'sensor_id'])
custom_pipeline.add_step('validate_records', rules=custom_rules)
custom_pipeline.add_step('custom_transformation', func=my_custom_function)

# Execute
result = custom_pipeline.fit_transform(df)
```

---

## 📥 Input Data Format

### Expected CSV Structure

```csv
timestamp,location_id,latitude,longitude,PM2.5,PM10,NO2,SO2,CO,O3,temperature,humidity,wind_speed
2024-01-01 00:00:00,LOC001,28.6139,77.2090,45.2,78.5,32.1,15.3,1.2,42.8,18.5,65.3,3.2
2024-01-01 01:00:00,LOC001,28.6139,77.2090,48.7,82.1,35.6,16.1,1.4,45.2,17.8,68.1,2.9
```

### Required Columns
- `timestamp`: Date and time of measurement
- `location_id`: Unique identifier for monitoring station
- `latitude`, `longitude`: GPS coordinates (WGS84)
- Pollutant columns: PM2.5, PM10, NO2, SO2, CO, O3, etc.
- Weather columns: temperature, humidity, wind_speed, pressure, etc.

---

## 📤 Output Data Format

### Feature-Engineered Dataset

```csv
timestamp,location_id,PM2.5_scaled,PM10_scaled,...,distance_to_road,distance_to_industrial,hour,day_of_week,season,hour_sin,hour_cos
2024-01-01 00:00:00,LOC001,0.45,0.62,...,120.5,2500.3,0,0,winter,0.0,1.0
```

### Output Files
```
data/processed/
├── final_feature_dataset.csv         # Main output
├── feature_scalers.pkl                # Saved scalers
├── feature_metadata.json              # Feature descriptions
└── data_quality_report.html           # Quality metrics
```

---

## ⚙️ Configuration

### pipeline_config.yaml

```yaml
data_cleaning:
  duplicates:
    subset: ['timestamp', 'location_id']
    keep: 'first'
  
  validation:
    PM2.5: [0, 500]
    PM10: [0, 1000]
    NO2: [0, 200]
    temperature: [-50, 60]
  
  missing_values:
    strategy: 'interpolate'
    max_gap: 6
    threshold: 0.3  # Drop features with >30% missing

  normalization:
    method: 'standard'
    features: ['PM2.5', 'PM10', 'NO2', 'temperature']

feature_engineering:
  spatial:
    enable: true
    buffer_radius: 500  # meters
    reference_data_path: 'data/spatial/'
  
  temporal:
    enable: true
    cyclical_encoding: true
    timezone: 'UTC'
```

---

## 🎯 Best Practices

1. **Data Quality First:** Always validate data before feature engineering
2. **Document Transformations:** Keep detailed logs of all cleaning steps
3. **Preserve Raw Data:** Never overwrite original datasets
4. **Version Control:** Track different versions of processed data
5. **Reproducibility:** Use random seeds and save preprocessing objects
6. **Domain Knowledge:** Validate ranges with environmental experts
7. **Iterative Approach:** Clean, analyze, refine in cycles

---

## 🐛 Troubleshooting

### Common Issues

**Issue:** High percentage of missing values
```python
# Solution: Analyze missing patterns
from diagnostics import analyze_missing_patterns
analysis = analyze_missing_patterns(df)
print(analysis['recommendations'])
```

**Issue:** Outliers affecting normalization
```python
# Solution: Use robust scaling
from sklearn.preprocessing import RobustScaler
scaler = RobustScaler()
df_scaled = scaler.fit_transform(df[features])
```

**Issue:** Spatial features calculation slow
```python
# Solution: Use spatial indexing
import geopandas as gpd
gdf = gpd.GeoDataFrame(df, geometry=gpd.points_from_xy(df.longitude, df.latitude))
gdf.sindex  # Creates spatial index for faster queries
```

---

## 📊 Example Output Statistics

After running the pipeline, expect:
- **Data Reduction:** 5-15% records removed (duplicates + invalid)
- **Missing Data:** <5% after imputation
- **Feature Count:** Original + 15-25 engineered features
- **Processing Time:** ~10-30 seconds per 100K records

---

## 📝 License

This project is licensed under the MIT License - see LICENSE file for details.

---


**Last Updated:** November 2025  
**Version:** 2.0.0

## Data Information
<img width="428" height="486" alt="image" src="https://github.com/user-attachments/assets/dbc498b5-671d-4930-bbc3-f1476ab5907e" />

