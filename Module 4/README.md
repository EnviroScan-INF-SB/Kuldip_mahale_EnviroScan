# 🌍 AI-EnviroScan: Module 4 – Model Training & Source Prediction

This module of **AI-EnviroScan** performs end-to-end machine-learning model training for **pollution-source prediction** using environmental, meteorological, and spatial data.

---

## ⚙️ Model Training Configuration

| Parameter | Value |
|------------|--------|
| **Random State** | 42 |
| **Test Size** | 0.2 |
| **Cross-Validation Folds (CV)** | 3 |

---

## 📥 Dataset Overview

**File Name:** `labeled_dataset_updated.csv`  
**Rows × Columns:** 10,800 × 46  
**File Size:** ≈ 3.44 MB  

### **Columns**
`sensor_id`, `sensor_name`, `sensor_latitude`, `sensor_longitude`, `area_type`, `measurement_timestamp`, `pollutant`, `pollutant_value`, `pollutant_unit`, `date`, `hour`, `is_weekend`, `season`, `timestamp_rounded`, `weather_timestamp`, `temperature_c`, `humidity_percent`, `pressure_hpa`, `wind_speed_ms`, `wind_direction_deg`, `precipitation_mm`, `weather_condition`, `visibility_km`, `road_edges`, `road_length_km`, `industrial_area`, `commercial_area`, `residential_area`, `green_space`, `water_body`, `educational`, `medical`, `transportation`, `building_density`, `aqi`, `pollution_category`, `traffic_influence`, `month`, `day_of_week`, `is_rush_hour`, **`PM2.5`**, **`PM10`**, **`NO2`**, **`CO`**, **`SO2`**, **`O3`**

---

## 🌫️ Added Pollutant Columns

| Column | Description | Typical Range (µg/m³ unless stated) |
|---------|--------------|------------------------------------|
| **PM2.5** | Fine particulate matter | 10 – 200 |
| **PM10** | Coarse particulate matter | 20 – 300 |
| **NO₂** | Nitrogen dioxide | 5 – 100 |
| **CO** | Carbon monoxide (mg/m³) | 0.1 – 10 |
| **SO₂** | Sulfur dioxide | 2 – 80 |
| **O₃** | Ozone | 10 – 150 |

All values were generated using realistic environmental distributions.

---

## 🎯 Target Variable: `pollution_source`

### **Created Categories & Distribution**
| Source Type | Samples | Percentage |
|--------------|----------|-------------|
| Industrial | 5,400 | 50 % |
| Residential | 2,160 | 20 % |
| Commercial | 2,160 | 20 % |
| Background | 1,080 | 10 % |

---

## 🧮 Feature Engineering & Selection

- **Total Features:** 25  
- **Included Features:**  
  `PM2.5`, `PM10`, `NO2`, `CO`, `SO2`, `O3`, `aqi`, `temperature_c`, `humidity_percent`, `wind_speed_ms`, plus 15 other contextual features.  
- **Feature Shape:** (10,800, 25)  
- **Target Shape:** (10,800,)  

---

## 🧠 Data Preprocessing

- Missing values handled: 7,560 → 0  
- Features scaled via `StandardScaler`  
- Target encoded as:  

- Train/Test Split: 8,640 / 2,160 samples  

---

## 🤖 Models Trained

| Model | Description |
|--------|--------------|
| **Random Forest Classifier** | Ensemble tree model for robust classification |
| **XGBoost Classifier** | Gradient-boosted decision trees for high accuracy |
| **Decision Tree Classifier** | Baseline model for interpretability |

---

## 🔧 Hyperparameter Tuning Results

| Model | Best Parameters | CV Accuracy |
|--------|----------------|-------------|
| **Random Forest** | `{'max_depth': 10, 'n_estimators': 100}` | 1.0000 |
| **XGBoost** | `{'learning_rate': 0.1, 'max_depth': 6, 'n_estimators': 100}` | 1.0000 |
| **Decision Tree** | `{'max_depth': 10}` | 1.0000 |

---

## 📈 Model Evaluation Results

| Model | Test Accuracy | CV Accuracy (±SD) | Precision | Recall | F1-Score |
|--------|---------------|-------------------|------------|---------|-----------|
| **Random Forest** | 1.0000 | 1.0000 ± 0.0000 | 1.0000 | 1.0000 | 1.0000 |
| **XGBoost** | 1.0000 | 1.0000 ± 0.0000 | 1.0000 | 1.0000 | 1.0000 |
| **Decision Tree** | 1.0000 | 1.0000 ± 0.0000 | 1.0000 | 1.0000 | 1.0000 |

---

## 🔍 Feature Importance & Visualization

- Feature-importance plots highlight pollutant-based features as dominant contributors.  
- Confusion matrices confirm perfect class separation across all categories.

---

## 💾 Exported Artifacts

| File Name | Description |
|------------|-------------|
| `pollution_source_random_forest_model.joblib` | Trained Random Forest model |
| `pollution_source_xgboost_model.joblib` | Trained XGBoost model |
| `pollution_source_decision_tree_model.joblib` | Trained Decision Tree model |
| `pollution_source_model_artifacts.joblib` | Combined artifact for deployment |

---

## 🏆 Final Report

| Metric | Best Model (Random Forest) |
|---------|----------------------------|
| **Accuracy** | 1.0000 |
| **F1-Score** | 1.0000 |
| **Best Parameters** | `{'max_depth': 10, 'n_estimators': 100}` |

All models achieved perfect classification on the current dataset.

---

## 📊 Tech Stack

- **Language:** Python 3.x  
- **Frameworks:** scikit-learn, XGBoost, pandas, numpy  
- **Environment:** Jupyter Notebook / Colab  

---

## 🚀 Next Steps

- Validate on new real-world datasets to check for overfitting.  
- Integrate the Random Forest model into the AI-EnviroScan dashboard for real-time source classification.  
- Expand feature set with additional geospatial and meteorological data.

---

## 📜 License

Released under the **MIT License**.  
Use, modify, and distribute freely with attribution.

---

**Developed as part of the AI-EnviroScan Pollution-Source Prediction Pipeline.**

