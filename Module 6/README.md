
# 📊 AI-EnviroScan: Module 6 – Real-Time Dashboard and Alerts

[![Python](https://img.shields.io/badge/Python-3.8%2B-blue.svg)](https://www.python.org/)
[![Streamlit](https://img.shields.io/badge/Streamlit-Latest-FF4B4B.svg)](https://streamlit.io/)
[![Plotly](https://img.shields.io/badge/Plotly-Latest-3F4F75.svg)](https://plotly.com/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

## 📘 Overview

This module provides a **real-time interactive dashboard** for comprehensive pollution monitoring across major Indian cities using **Streamlit**. It integrates all previously trained models and visualization modules into a single, user-friendly interface — delivering **live insights, AI-powered predictions, and intelligent alerts** for environmental decision-making.

The dashboard empowers:
- 🏛️ **Government Agencies** - Monitor and respond to pollution events
- 🏥 **Health Departments** - Track air quality impacts on public health
- 🔬 **Researchers** - Analyze pollution patterns and trends
- 👥 **Citizens** - Stay informed about local air quality

---

## ⚙️ Key Features

| Feature | Description |
|---------|-------------|
| ✅ **Interactive UI** | Built with Streamlit for intuitive user experience |
| ✅ **Multi-City Support** | Covers 20+ major Indian cities across zones |
| ✅ **AI Predictions** | Real-time pollution source identification with confidence scores |
| ✅ **Smart Alerts** | Automated warnings when pollutants exceed safe thresholds |
| ✅ **Trend Analysis** | Interactive charts showing temporal pollution patterns |
| ✅ **Geospatial View** | Live heatmap visualization of city-level pollution |
| ✅ **Report Generation** | Downloadable daily/weekly reports in CSV format |
| ✅ **Notifications** | Optional email/SMS alerts for critical conditions |

---

## 🧩 Dashboard Workflow

### 1️⃣ Data Loading and Initialization

The system automatically loads pollution data from multiple sources:

```python
import pandas as pd
import streamlit as st

class PollutionDashboard:
    def __init__(self):
        # Load enhanced pollution dataset
        self.df = pd.read_csv('data/pollution_data_all_india_enhanced.csv')
        self.df['timestamp'] = pd.to_datetime(self.df['timestamp'])
        
        # Load trained ML model
        self.model = joblib.load('models/pollution_source_model.joblib')
```

**Data Sources:**
- Enhanced pollution datasets from government APIs
- Historical data archives
- Real-time sensor networks
- Fallback synthetic data for demo purposes

**Covered Cities (20+ locations):**
- **North**: Delhi, Chandigarh, Jaipur, Lucknow
- **South**: Bangalore, Chennai, Hyderabad, Thiruvananthapuram
- **East**: Kolkata, Bhubaneswar, Guwahati, Patna
- **West**: Mumbai, Pune, Ahmedabad, Surat
- **Central**: Bhopal, Indore, Nagpur, Raipur

---

### 2️⃣ User Input Interface

Flexible input options through an intuitive sidebar:

```python
def create_sidebar_inputs(self):
    st.sidebar.header("🌍 Location Selection")
    
    # Zone selection
    zone = st.sidebar.selectbox(
        "Select Zone",
        ["North India", "South India", "East India", "West India", "Central India"]
    )
    
    # City selection based on zone
    cities = self.get_cities_by_zone(zone)
    selected_city = st.sidebar.selectbox("Select City", cities)
    
    # Input method
    input_method = st.sidebar.radio(
        "Input Mode",
        ["🛰️ Real-Time Data", "✏️ Manual Input", "📅 Historical Analysis"]
    )
    
    return selected_city, lat, lon, pollution_data, input_method
```

**Input Modes:**

1. **🛰️ Real-Time Data** - Fetch live sensor readings
2. **✏️ Manual Input** - Custom pollutant values for simulation
3. **📅 Historical Analysis** - Explore past pollution patterns

**Manual Input Parameters:**
- PM2.5, PM10 (particulate matter)
- NO₂, SO₂ (nitrogen and sulfur dioxide)
- CO (carbon monoxide)
- O₃ (ozone)
- Temperature, humidity, wind speed

---

### 3️⃣ AI-Based Source Prediction

Intelligent pollution source identification using trained ML models:

```python
def predict_pollution_source(self, pollution_data):
    """
    Predict pollution source using trained ML model
    Returns: predicted sources and confidence probabilities
    """
    # Prepare features
    features = self.prepare_features(pollution_data)
    
    # Get prediction and probabilities
    prediction = self.model.predict(features)
    probabilities = self.model.predict_proba(features)
    
    # Map to source categories
    source_labels = ['Industrial', 'Vehicular', 'Construction', 
                     'Agricultural', 'Natural']
    
    results = dict(zip(source_labels, probabilities[0]))
    return prediction[0], results
```

**Source Categories:**
- 🏭 **Industrial** - Factory emissions, manufacturing
- 🚗 **Vehicular** - Traffic, transportation
- 🏗️ **Construction** - Building activities, dust
- 🌾 **Agricultural** - Crop burning, farming
- 🌿 **Natural** - Dust storms, wildfires

**Confidence Display:**
```python
# Display prediction results
st.subheader("🎯 AI Source Prediction")
st.metric("Primary Source", primary_source, delta=f"{confidence:.1f}% confidence")

# Probability bars
for source, prob in probabilities.items():
    st.progress(prob)
    st.write(f"{source}: {prob*100:.1f}%")

# Pie chart visualization
fig = px.pie(values=list(probabilities.values()), 
             names=list(probabilities.keys()),
             title="Source Probability Distribution")
st.plotly_chart(fig)
```

---

### 4️⃣ Real-Time Alert System

Continuous monitoring with intelligent threshold-based alerts:

```python
def check_pollution_alerts(self, pollution_data):
    """
    Check pollutant levels against safety thresholds
    Returns: list of alert messages with severity levels
    """
    alerts = []
    
    # Define thresholds (EPA/WHO standards)
    thresholds = {
        'PM2.5': {'good': 12, 'moderate': 35, 'unhealthy': 55, 'hazardous': 150},
        'PM10': {'good': 54, 'moderate': 154, 'unhealthy': 254, 'hazardous': 424},
        'NO2': {'good': 40, 'moderate': 100, 'unhealthy': 360, 'hazardous': 649},
        'SO2': {'good': 20, 'moderate': 80, 'unhealthy': 250, 'hazardous': 500},
        'CO': {'good': 1, 'moderate': 2, 'unhealthy': 10, 'hazardous': 17},
        'O3': {'good': 54, 'moderate': 70, 'unhealthy': 85, 'hazardous': 105}
    }
    
    # Check each pollutant
    for pollutant, value in pollution_data.items():
        if value > thresholds[pollutant]['hazardous']:
            alerts.append({
                'pollutant': pollutant,
                'value': value,
                'level': 'HAZARDOUS',
                'color': 'red',
                'message': f'🚨 CRITICAL: {pollutant} level extremely high!'
            })
        elif value > thresholds[pollutant]['unhealthy']:
            alerts.append({
                'pollutant': pollutant,
                'value': value,
                'level': 'UNHEALTHY',
                'color': 'orange',
                'message': f'⚠️ WARNING: {pollutant} level unhealthy!'
            })
        elif value > thresholds[pollutant]['moderate']:
            alerts.append({
                'pollutant': pollutant,
                'value': value,
                'level': 'MODERATE',
                'color': 'yellow',
                'message': f'⚡ MODERATE: {pollutant} level elevated.'
            })
    
    return alerts
```

**Air Quality Index (AQI) Standards:**

| Pollutant | Good | Moderate | Unhealthy | Hazardous |
|-----------|------|----------|-----------|-----------|
| **PM2.5** | ≤12 µg/m³ | ≤35 µg/m³ | ≤55 µg/m³ | >150 µg/m³ |
| **PM10** | ≤54 µg/m³ | ≤154 µg/m³ | ≤254 µg/m³ | >424 µg/m³ |
| **NO₂** | ≤40 µg/m³ | ≤100 µg/m³ | ≤360 µg/m³ | >649 µg/m³ |
| **SO₂** | ≤20 µg/m³ | ≤80 µg/m³ | ≤250 µg/m³ | >500 µg/m³ |
| **CO** | ≤1 mg/m³ | ≤2 mg/m³ | ≤10 mg/m³ | >17 mg/m³ |
| **O₃** | ≤54 µg/m³ | ≤70 µg/m³ | ≤85 µg/m³ | >105 µg/m³ |

**Alert Display:**
```python
# Display alerts with color coding
for alert in alerts:
    if alert['level'] == 'HAZARDOUS':
        st.error(alert['message'])
    elif alert['level'] == 'UNHEALTHY':
        st.warning(alert['message'])
    else:
        st.info(alert['message'])
```

---

### 5️⃣ Pollution Trends Visualization

Interactive time-series analysis with dynamic charts:

```python
def display_pollution_trends(self, city, days=7):
    """
    Display pollutant trends over specified time period
    """
    st.subheader(f"📈 Pollution Trends - Last {days} Days")
    
    # Filter data for selected city and time range
    end_date = datetime.now()
    start_date = end_date - timedelta(days=days)
    
    city_data = self.df[
        (self.df['city'] == city) & 
        (self.df['timestamp'] >= start_date)
    ]
    
    # Create multi-line chart
    fig = go.Figure()
    
    pollutants = ['PM2.5', 'PM10', 'NO2', 'SO2', 'CO', 'O3']
    for pollutant in pollutants:
        fig.add_trace(go.Scatter(
            x=city_data['timestamp'],
            y=city_data[pollutant],
            mode='lines+markers',
            name=pollutant,
            line=dict(width=2)
        ))
    
    # Add threshold lines
    fig.add_hline(y=35, line_dash="dash", line_color="orange", 
                  annotation_text="PM2.5 Moderate")
    fig.add_hline(y=150, line_dash="dash", line_color="red", 
                  annotation_text="PM2.5 Hazardous")
    
    fig.update_layout(
        title=f"Pollutant Concentrations - {city}",
        xaxis_title="Date",
        yaxis_title="Concentration (µg/m³)",
        hovermode='x unified',
        height=500
    )
    
    st.plotly_chart(fig, use_container_width=True)
```

**Visualization Features:**
- Multi-pollutant comparison
- Threshold reference lines
- Interactive hover details
- Zoom and pan capabilities
- Time range selection (7/14/30 days)

---

### 6️⃣ Interactive Geospatial Map

Real-time pollution visualization with Plotly Mapbox:

```python
def create_pollution_map(self, zone=None):
    """
    Create interactive map showing pollution levels across cities
    """
    st.subheader("🗺️ Live Pollution Map")
    
    # Get latest data for all cities
    latest_data = self.df.groupby('city').last().reset_index()
    
    # Filter by zone if specified
    if zone:
        latest_data = latest_data[latest_data['zone'] == zone]
    
    # Create scatter mapbox
    fig = px.scatter_mapbox(
        latest_data,
        lat="sensor_latitude",
        lon="sensor_longitude",
        color="PM2.5",
        size="PM2.5",
        hover_name="city",
        hover_data={
            'PM2.5': ':.1f',
            'PM10': ':.1f',
            'NO2': ':.1f',
            'AQI': ':.0f'
        },
        color_continuous_scale=[
            [0, 'green'],      # Good
            [0.3, 'yellow'],   # Moderate
            [0.6, 'orange'],   # Unhealthy
            [1, 'red']         # Hazardous
        ],
        size_max=30,
        zoom=4,
        mapbox_style="open-street-map",
        title="Real-Time Pollution Levels"
    )
    
    fig.update_layout(height=600, margin={"r":0,"t":40,"l":0,"b":0})
    st.plotly_chart(fig, use_container_width=True)
```

**Map Features:**
- 🎨 Color-coded pollution levels (green → yellow → orange → red)
- 📍 City markers with hover information
- 🔍 Zoom and pan functionality
- 📊 Size proportional to pollution intensity
- 🗺️ Multiple map styles (street, satellite, terrain)

---

### 7️⃣ Alert Notification System

Subscription-based alert delivery:

```python
def setup_alert_notifications(self):
    """
    Configure alert notification preferences
    """
    st.subheader("🔔 Alert Notifications")
    
    col1, col2 = st.columns(2)
    
    with col1:
        email = st.text_input("📧 Email Address", placeholder="your.email@example.com")
        enable_email = st.checkbox("Enable email alerts")
    
    with col2:
        phone = st.text_input("📱 Phone Number", placeholder="+91-XXXXXXXXXX")
        enable_sms = st.checkbox("Enable SMS alerts")
    
    # Alert preferences
    st.write("**Alert Triggers:**")
    alert_aqi = st.slider("Alert when AQI exceeds:", 0, 500, 150)
    alert_pm25 = st.slider("Alert when PM2.5 exceeds (µg/m³):", 0, 200, 55)
    
    # Subscribe button
    if st.button("🔔 Subscribe to Alerts"):
        if enable_email or enable_sms:
            # Save preferences (implement backend logic)
            self.save_alert_preferences(email, phone, enable_email, enable_sms)
            st.success("✅ You'll receive notifications for critical pollution conditions!")
        else:
            st.warning("Please enable at least one notification method.")
```

**Notification Features:**
- 📧 Email alerts with detailed reports
- 📱 SMS notifications for critical events
- ⚙️ Customizable threshold settings
- ⏰ Configurable alert frequency
- 🔕 Easy unsubscribe options

---

### 8️⃣ Report Generation and Export

Comprehensive reporting with downloadable formats:

```python
def generate_pollution_report(self, city, report_type='daily'):
    """
    Generate pollution report for specified period
    """
    st.subheader("📊 Report Generation")
    
    # Report type selection
    report_type = st.selectbox(
        "Report Type",
        ["Daily Summary", "Weekly Analysis", "Monthly Overview", "Custom Range"]
    )
    
    # Generate report data
    if report_type == "Daily Summary":
        report_data = self.generate_daily_report(city)
    elif report_type == "Weekly Analysis":
        report_data = self.generate_weekly_report(city)
    else:
        report_data = self.generate_monthly_report(city)
    
    # Display summary statistics
    st.write("**Summary Statistics:**")
    col1, col2, col3 = st.columns(3)
    
    with col1:
        st.metric("Average PM2.5", f"{report_data['pm25_avg']:.1f} µg/m³")
    with col2:
        st.metric("Peak AQI", f"{report_data['aqi_max']:.0f}")
    with col3:
        st.metric("Alerts Triggered", report_data['alert_count'])
    
    # Convert to CSV
    csv_data = report_data['dataframe'].to_csv(index=False)
    
    # Download button
    st.download_button(
        label="📥 Download Report (CSV)",
        data=csv_data,
        file_name=f"pollution_report_{city}_{report_type}_{datetime.now().strftime('%Y%m%d')}.csv",
        mime="text/csv"
    )
    
    # Optional: Generate PDF report
    if st.button("📄 Generate PDF Report"):
        pdf_data = self.generate_pdf_report(report_data)
        st.download_button(
            label="📥 Download PDF",
            data=pdf_data,
            file_name=f"pollution_report_{city}.pdf",
            mime="application/pdf"
        )
```

**Report Contents:**
- 📊 Statistical summaries (mean, median, max, min)
- 📈 Trend analysis with visualizations
- 🚨 Alert history and frequency
- 🎯 Source attribution breakdown
- 📍 Geospatial hotspot identification
- 💡 Recommendations and insights

---

## 📊 Dashboard Tabs

The dashboard is organized into intuitive tabs for different functionalities:

| Tab | Icon | Description |
|-----|------|-------------|
| **AI Analysis** | 🎯 | Pollution source prediction with confidence scores |
| **Trends** | 📈 | Time-series analysis and historical patterns |
| **Map View** | 🗺️ | Interactive geospatial visualization |
| **Alerts** | 🚨 | Real-time warnings and notification setup |
| **Reports** | 📊 | Generate and download comprehensive reports |
| **Settings** | ⚙️ | Configure preferences and thresholds |

### Tab Implementation:

```python
def main():
    st.set_page_config(page_title="AI-EnviroScan Dashboard", 
                       page_icon="🌍", 
                       layout="wide")
    
    dashboard = PollutionDashboard()
    
    # Create tabs
    tab1, tab2, tab3, tab4, tab5 = st.tabs([
        "🎯 AI Analysis", 
        "📈 Trends", 
        "🗺️ Map", 
        "🚨 Alerts", 
        "📊 Reports"
    ])
    
    with tab1:
        dashboard.display_ai_analysis()
    
    with tab2:
        dashboard.display_trends()
    
    with tab3:
        dashboard.display_map()
    
    with tab4:
        dashboard.display_alerts()
    
    with tab5:
        dashboard.display_reports()
```

---

## 🛠️ Installation

### Prerequisites

- Python 3.8 or higher
- pip package manager
- Internet connection (for map tiles)

### Dependencies

Install all required packages:

```bash
pip install streamlit pandas numpy plotly joblib scikit-learn folium
```

Or use the requirements file:

```bash
pip install -r requirements.txt
```

**requirements.txt:**
```
streamlit>=1.28.0
pandas>=1.3.0
numpy>=1.21.0
plotly>=5.14.0
joblib>=1.0.0
scikit-learn>=1.0.0
folium>=0.12.0
requests>=2.28.0
python-dateutil>=2.8.2
```

---

## 🚀 Running the Dashboard

### Local Deployment

```bash
# Navigate to project directory
cd AI-EnviroScan

# Run the dashboard
streamlit run streamlit_dashboard.py
```

The dashboard will automatically open in your default browser at:
```
http://localhost:8501
```

### Configuration Options

```bash
# Custom port
streamlit run streamlit_dashboard.py --server.port 8080

# Custom address
streamlit run streamlit_dashboard.py --server.address 0.0.0.0

# Enable CORS
streamlit run streamlit_dashboard.py --server.enableCORS false
```

### Cloud Deployment

#### Streamlit Cloud:
```bash
# Push to GitHub
git push origin main

# Deploy on streamlit.io
# Connect repository and deploy
```

#### Docker:
```dockerfile
FROM python:3.9-slim

WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

EXPOSE 8501

CMD ["streamlit", "run", "streamlit_dashboard.py", "--server.port=8501", "--server.address=0.0.0.0"]
```

---

## 📁 Project Structure

```
AI-EnviroScan/
├── AI-EnviroScan/
│   ├── add_all_india_cities_enhanced.py    # Script to enhance city data
│   ├── AI-EnviroScan_Complete_Dashboard.ipynb  # Complete dashboard notebook
│   ├── Final_Dashboard.ipynb               # Final integrated dashboard
│   ├── streamlit_dashboard.py              # Streamlit web application
│   ├── unified_dashboard.py                # Unified dashboard module
│   ├── requirements.txt                    # Python dependencies
│   │
│   ├── data/                               # Pollution datasets
│   │   ├── pollution_data_all_india.csv
│   │   ├── pollution_data_all_india_enhanced.csv
│   │   ├── pollution_data_comprehensive.csv
│   │   └── pollution_data_simple.csv
│   │
│   ├── exports/                            # Generated reports and summaries
│   │   ├── data_collection_summary.json
│   │   ├── model_performance_report.json
│   │   ├── pollution_statistics_report.csv
│   │   ├── prediction_accuracy_report.csv
│   │   ├── sensor_summary_report.csv
│   │   └── map_links_report.csv
│   │
│   ├── maps/                               # Interactive HTML maps
│   │   ├── comprehensive_dashboard_map.html
│   │   ├── pm25_heatmap_map.html
│   │   ├── pm10_heatmap_map.html
│   │   ├── no2_heatmap_map.html
│   │   ├── pollution_source_predictions.html
│   │   ├── risk_zones_map.html
│   │   ├── sensor_markers_map.html
│   │   └── time_animation_map.html
│   │
│   ├── models/                             # Trained ML models
│   │   ├── pollution_source_decision_tree_model.joblib
│   │   ├── pollution_source_random_forest_model.joblib
│   │   ├── pollution_source_xgboost_model.joblib
│   │   ├── pollution_source_model_artifacts.joblib
│   │   └── preprocessing_artifacts.joblib
│   │
│   ├── notebooks/                          # Jupyter notebooks for development
│   │   ├── 01_Data_Collection.ipynb
│   │   ├── 02_Model_Training.ipynb
│   │   ├── 03_Mapping_Visualization.ipynb
│   │   ├── 04_Dashboard_Preview.ipynb
│   │   └── cache/                          # Notebook cache files
│   │       └── *.json
│   │
│   ├── __pycache__/                        # Python cache files
│   │   └── unified_dashboard.cpython-311.pyc
│   │
│   └── README.md                           # This file
```

### 📂 Directory Descriptions

| Directory | Purpose |
|-----------|---------|
| **data/** | Contains all pollution datasets in various formats (simple, comprehensive, enhanced) |
| **exports/** | Stores generated reports, summaries, and analysis results in JSON/CSV formats |
| **maps/** | Interactive HTML maps for different pollutants and visualizations |
| **models/** | Trained machine learning models (Decision Tree, Random Forest, XGBoost) and preprocessing artifacts |
| **notebooks/** | Jupyter notebooks for development, training, and visualization workflows |
| **__pycache__/** | Python bytecode cache (auto-generated) |

---

## ⚙️ Configuration

### Settings File (config/settings.json):

```json
{
  "thresholds": {
    "pm25": {"good": 12, "moderate": 35, "unhealthy": 55, "hazardous": 150},
    "pm10": {"good": 54, "moderate": 154, "unhealthy": 254, "hazardous": 424}
  },
  "alert_settings": {
    "email_enabled": true,
    "sms_enabled": false,
    "alert_frequency": "hourly"
  },
  "map_settings": {
    "default_zoom": 5,
    "map_style": "open-street-map"
  }
}
```

---

## 🎨 Customization

### Custom Themes

```python
# .streamlit/config.toml
[theme]
primaryColor = "#FF4B4B"
backgroundColor = "#FFFFFF"
secondaryBackgroundColor = "#F0F2F6"
textColor = "#262730"
font = "sans serif"
```

### Custom Components

```python
# Add custom CSS
st.markdown("""
    <style>
    .metric-card {
        background-color: #f0f2f6;
        padding: 20px;
        border-radius: 10px;
        box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    }
    </style>
""", unsafe_allow_html=True)
```

---

## 📈 Performance Optimization

### Caching Strategies

```python
@st.cache_data(ttl=3600)  # Cache for 1 hour
def load_pollution_data():
    return pd.read_csv('data/pollution_data.csv')

@st.cache_resource
def load_ml_model():
    return joblib.load('models/pollution_source_model.joblib')
```

### Lazy Loading

```python
# Load data only when needed
if st.session_state.get('data_loaded') is None:
    st.session_state.data = load_pollution_data()
    st.session_state.data_loaded = True
```

---

## 🤝 Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/dashboard-enhancement`)
3. Commit your changes (`git commit -am 'Add new dashboard feature'`)
4. Push to the branch (`git push origin feature/dashboard-enhancement`)
5. Open a Pull Request

---

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## 👥 Authors

**AI-EnviroScan Team**

---


## 🔗 Related Modules

- **Module 1**: Data Collection and Integration
- **Module 2**: Data Cleaning and Preprocessing
- **Module 3**: Feature Engineering
- **Module 4**: Model Training and Source Prediction
- **Module 5**: Geospatial Mapping and Visualization
- **Module 6**: Real-Time Dashboard (This Module)

---

## 📚 Resources

- [Streamlit Documentation](https://docs.streamlit.io/)
- [Plotly Documentation](https://plotly.com/python/)
- [Air Quality Standards](https://cpcb.nic.in/)
- [WHO Air Quality Guidelines](https://www.who.int/airpollution/guidelines/)

---

**Last Updated**: November 2025  
**Version**: 1.0.0  
**Status**: Production Ready 🚀
