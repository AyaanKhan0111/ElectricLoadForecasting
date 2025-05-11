# Electric Load Forecasting Using Data Mining Techniques

**Authors**: Muhammad Qasim (22I-1994), Ayaan Khan (22I-2066), Abu Bakr Nadeem (22I-2003)

## Project Overview

Electric load forecasting plays a vital role in energy management, grid reliability, and demand response. In this project, we perform an end-to-end data mining and machine learning workflow on an hourly electricity demand dataset fused with weather measurements for ten major U.S. cities.

### Objectives

1. **Cluster Analysis**: Uncover latent patterns by grouping observations with similar demand–weather characteristics, enabling tailored demand-response strategies.
2. **Predictive Modeling**: Develop and compare multiple forecasting algorithms—including statistical, machine-learning, and deep-learning models—to accurately predict next-day hourly electricity demand.
3. **Front-End Interface**: Create an interactive web application for selecting cities, adjusting model parameters, and visualizing clustering results and forecast outputs in real time.

## Table of Contents

1. [Dataset Description](#dataset-description)
2. [Data Preprocessing](#data-preprocessing)
3. [Cluster Analysis](#cluster-analysis)
4. [Predictive Modeling](#predictive-modeling)
5. [Front-End Interface](#front-end-interface)
6. [Results and Conclusions](#results-and-conclusions)
7. [Future Work](#future-work)
8. [Usage](#usage)
9. [License](#license)

## Dataset Description

The project dataset comprises two parts:

1. **Weather Data**: Ten JSON files (one per city) containing hourly readings for:

   * `temperature` (°F)
   * `humidity` (%)
   * `windSpeed` (mph)
   * `pressure` (hPa)
   * `dewPoint` (°F)
   * Timestamp as UNIX epoch seconds
     Cities: Dallas, Houston, Los Angeles, New York City, San Diego, San Jose, San Antonio, Phoenix, Philadelphia, Seattle.

2. **Demand Data**: Three CSV files covering six regions’ hourly electricity demand in megawatt-hours (MWh). File mappings:

   ```python
   DEMAND_FILES = {
       'nyc': 'cleaned_subregion_data.csv',
       'phoenix': 'cleaned_balance_data.csv',
       'seattle': 'cleaned_balance_data.csv',
       'houston': 'cleaned_texas_data.csv',
       'san antonio': 'cleaned_texas_data.csv',
       'dallas': 'cleaned_texas_data.csv'
   }
   ```

All datasets are merged on `timestamp` and `city` into a single table, yielding features for time, weather, and demand. After merging, there are approximately *N* records per city per hour.

**Schema**:

| Column      | Type        | Description                     |
| ----------- | ----------- | ------------------------------- |
| timestamp   | datetime    | Hourly timestamp (UTC)          |
| city        | categorical | City name                       |
| temperature | float       | Ambient temperature (°F)        |
| humidity    | float       | Relative humidity (%)           |
| windSpeed   | float       | Wind speed (mph)                |
| pressure    | float       | Atmospheric pressure (hPa)      |
| dewPoint    | float       | Dew point temperature (°F)      |
| demand\_mwh | float       | Hourly electricity demand (MWh) |

## Data Preprocessing

1. **Loading & Inspection**

   * Parsed JSONs and CSVs; converted UNIX epoch to datetime.
   * Melted wide-format Texas CSVs to long-form for consistency.
   * Merged on `(timestamp, city)` preserving all weather entries.

2. **Missing Values**

   * Detected \~3% nulls in weather features; removed records missing `demand_mwh`.
   * Dropped remaining null weather entries to maintain data integrity.

3. **Feature Engineering**

   * Extracted `hour`, `day_of_week`, `month`, and `season` (via month-to-season map).
   * Applied `RobustScaler` to continuous variables.

4. **Aggregation**

   * Computed weekly summaries by city: mean temperature, humidity, wind speed, and demand statistics.
   * Exported as `weekly_summary.csv`.

5. **Anomaly Detection**

   * Trained `IsolationForest` (1% contamination) on weather features.
   * Flagged anomalies, imputed outliers with feature-wise medians.

Cleaned data saved to `preprocessed_and_cleaned_data.csv`.

## Cluster Analysis

### Objective

Segment hourly observations into clusters based on combined weather–demand profiles.

### Methodology

1. Selected features: temperature, humidity, windSpeed, pressure, dewPoint, demand\_mwh.
2. Scaled with `RobustScaler`.
3. Reduced to 2 principal components via PCA for visualization.

**Elbow Method**: Evaluated SSE for k = 1–9 and identified elbow at k = 3.

**Results**:

* **k = 3** clusters with silhouette score 0.284.

| Cluster | Label                        | Interpretation                                          |
| ------- | ---------------------------- | ------------------------------------------------------- |
| 0       | Cool Midday, Moderate Demand | Afternoon hours with mild temperatures and steady load. |
| 1       | Hot Afternoon, High Demand   | Peak summer afternoons driving highest usage.           |
| 2       | Humid Morning, Low Demand    | Early hours with high humidity but lower load.          |

**Insights**:

* Cluster 1 aligns with peak cooling loads; dynamic pricing may alleviate stress.
* Cluster 2 represents off-peak windows, ideal for maintenance scheduling.

## Predictive Modeling

### Objective

Forecast next 24 hours of electricity demand per city using historical and weather data.

### Models

1. Naïve baseline (previous day's same hour).
2. Linear Regression with hyperparameter tuning.
3. SARIMA (seasonal ARIMA) for city-level series.
4. XGBoost with grid search optimization.
5. LSTM neural network on full feature set.

### Training & Validation

* Chronological 80/20 split; no leakage.
* 5-fold CV for regression, 3-fold for XGBoost.
* Scaling and one-hot encoding for categorical features.

### Metrics

* MAE, RMSE, MAPE, R².

### Summary

| Model   | R²     | MAE    | RMSE   | MAPE (%) |
| ------- | ------ | ------ | ------ | -------- |
| Linear  | 0.8763 | 0.1295 | 0.1816 | —        |
| XGBoost | 0.9360 | 0.0729 | 0.1307 | —        |
| LSTM    | 0.9648 | 0.0663 | 0.1137 | —        |
| SARIMA  | 0.8528 | 155.34 | 338.94 | 4.38     |

Ensemble of XGBoost + LSTM reduced RMSE by \~2% over individual models.

## Front-End Interface

A Flask backend serves preprocess data and model artifacts to a web frontend. Features:

1. **Parameter Selection**: city dropdown, date range picker, model checkboxes, cluster slider.
2. **Visualizations**: interactive PCA scatter, forecast time-series with confidence bands.
3. **Export**: download forecast CSV.
4. **Help**: inline tooltips and metric definitions.

## Results and Conclusions

Based on evaluation across all cities:

* **LSTM**: highest accuracy (R² ≈ 0.965, MAE ≈ 0.066).
* **XGBoost**: strong balance of performance and interpretability (R² ≈ 0.936).
* **Ensemble**: most stable forecasts, slight RMSE improvement.

**Recommendation**: Deploy combined LSTM + XGBoost ensemble for robust next-day forecasting.

## Future Work

* Incorporate additional meteorological variables (precipitation, solar radiation) and socioeconomic factors.
* Explore Transformer-based time-series models (e.g., Temporal Fusion Transformers).
* Implement real-time data pipelines and continuous model updates.
* Investigate spatial–temporal clustering across cities for regional grid management.

## Usage

1. Clone the repository:

   ```bash
   git clone <repository_url>
   cd <repo_directory>
   ```
2. Install dependencies:

   ```bash
   pip install -r requirements.txt
   ```
3. Run preprocessing:

   ```bash
   python preprocess.py --weather_dir data/weather --demand_dir data/demand
   ```
4. Train models:

   ```bash
   python train.py --config config.yaml
   ```
5. Launch web application:

   ```bash
   python app.py
   ```

## License

This project is licensed under the MIT License. See LICENSE file for details.
