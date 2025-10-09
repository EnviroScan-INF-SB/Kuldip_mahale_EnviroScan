# 🌍 EnviroScan: AI-Powered Pollution Source Identifier using Geospatial Analytics

## 🚀 Project Overview
EnviroScan leverages **Machine Learning** 🤖 and **Geospatial Analytics** 📍 to monitor pollution levels and predict likely pollution sources such as industrial 🏭, vehicular 🚗, agricultural 🚜, or natural 🍃 origins. By combining real-time sensor data, weather parameters 🌦️, and location-based features, EnviroScan supports targeted environmental interventions and data-driven policy making 📊.

---

## ✨ Key Features
- 🔍 **AI-based pollution source prediction** using advanced machine learning models.
- 🗺️ **Real-time pollution hotspot mapping** and risk zone visualization on interactive maps.
- 🚨 **Automated pollution alerts** triggered when pollution thresholds or source confidence levels are exceeded.
- 🏙️ Support for **data-driven policy making** and urban planning initiatives.
- 📈 **Comprehensive reports and visualizations** tailored for environmental agencies and stakeholders.

---

## 📅 Project Modules & Timeline

### Milestone 1: Data Acquisition & Cleaning (Week 1–2)
- **Data Collection**  
  - Gather air quality data (PM2.5, PM10, NO₂, CO, SO₂, O₃) from the [OpenAQ API](https://openaq.org/) for selected locations.  
  - Collect weather data (temperature, humidity, wind speed, wind direction) via the [OpenWeatherMap API](https://openweathermap.org/api).  
  - Extract nearby physical features (roads 🛣️, industrial zones 🏭, dump sites 🚮, agricultural fields 🌾) using [OpenStreetMap](https://www.openstreetmap.org/) with the OSMnx Python package.  
- **Data Tagging & Storage**  
  - Tag each data point with latitude, longitude, timestamp ⏰, and source metadata.  
  - Store the data in structured CSV/JSON formats for easy preprocessing and modeling.
