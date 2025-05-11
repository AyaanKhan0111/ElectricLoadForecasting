Electric Load Forecasting Using Data Mining Techniques
Authors
Muhammad Qasim (22I-1994)
Ayaan Khan (22I-2066)
Abu Bakr Nadeem (22I-2003)

1. Project Overview
Electric load forecasting is essential for effective energy management, grid reliability, and demand response planning. This project implements a comprehensive data mining and machine learning pipeline using hourly electricity demand data fused with weather measurements from ten major U.S. cities. Key objectives include:

Cluster Analysis: Identify patterns based on demand–weather characteristics for informed strategy development.

Predictive Modeling: Compare forecasting algorithms (statistical, machine-learning, and deep-learning) to predict next-day hourly electricity demand.

Interactive Front-End: Develop a web application for model selection, parameter tuning, and visualization.

2. Dataset Description
The dataset consists of:

Weather Data: Hourly weather features (temperature, humidity, wind speed, pressure, dew point) in JSON format for 10 cities.

Demand Data: Hourly electricity demand in MWh across six regions.

Cities covered: Dallas, Houston, Los Angeles, New York City, San Diego, San Jose, San Antonio, Phoenix, Philadelphia, Seattle.

Data was merged on (timestamp, city) into a unified dataset with time, weather, and demand features.

Schema
Column	Type	Description
timestamp	datetime	Hourly timestamp (UTC)
city	categorical	City name
temperature	float	Ambient temperature (°F)
humidity	float	Relative humidity (%)
windSpeed	float	Wind speed (mph)
pressure	float	Atmospheric pressure (hPa)
dewPoint	float	Dew point temperature (°F)
demand_mwh	float	Hourly electricity demand (MWh)

3. Data Preprocessing
3.1 Loading & Merging
Parsed weather JSONs and converted UNIX timestamps.

Cleaned and reshaped regional demand CSVs.

Merged datasets on (timestamp, city).

3.2 Missing Values
~3% nulls mainly in weather data.

Dropped rows with missing demand; imputed weather nulls using median after anomaly detection.

3.3 Feature Engineering
Extracted: hour, day_of_week, month, season.

Applied RobustScaler for outlier-insensitive scaling.

3.4 Aggregation
Weekly summaries computed by city:

Mean weather metrics

Mean and total electricity demand

3.5 Anomaly Detection
Used IsolationForest (1% contamination) for detecting outliers.

Imputed flagged values with feature-wise medians.

Cleaned data saved to preprocessed_and_cleaned_data.csv.

4. Cluster Analysis
4.1 Goal
Group hourly records by joint weather–demand profiles for scenario-based strategy design.

4.2 Method
Selected six features.

Applied PCA for dimensionality reduction.

Used K-Means (k=3) determined via Elbow Method.

Clustering visualized using 2D PCA plot.

4.3 Cluster Interpretations
Cluster	Description
0	Cool midday, moderate demand
1	Hot afternoon, high demand
2	Humid morning, low demand

Silhouette Score: 0.284

5. Predictive Modeling
5.1 Objective
Forecast the next 24 hours of electricity demand per city using historical trends, weather, and time-based features.

5.2 Models Implemented
Naïve Baseline: Previous day’s demand

Linear Regression

SARIMA (e.g., Phoenix)

XGBoost

LSTM Neural Network

5.3 Training Strategy
Chronological 80/20 train-test split

Scaled numerical and encoded categorical variables

Hyperparameter tuning via cross-validation

5.4 Evaluation Metrics
Mean Absolute Error (MAE)

Root Mean Squared Error (RMSE)

Mean Absolute Percentage Error (MAPE)

R² Score

5.5 Results Summary
Model	R²	MAE	RMSE	MAPE (%)
Linear Regression	0.8763	0.1295	0.1816	-
XGBoost	0.9360	0.0729	0.1307	-
LSTM	0.9648	0.0663	0.1137	-
SARIMA (Phoenix)	0.8528	155.34	338.94	4.38

Model Insights:

LSTM performed best overall (R² ≈ 0.965).

XGBoost balanced performance and interpretability.

Linear and SARIMA models useful as benchmarks.

5.6 Ensemble Learning
Simple average of LSTM + XGBoost improved RMSE by ~2% on validation data.

6. Front-End Interface
Built with Flask (backend) and HTML/CSS/JS (frontend). Features include:

Controls:

City, date range, model selection

Clustering k-value slider

Visualizations:

PCA cluster scatter plot

Forecast line charts with confidence intervals

Export Options:

Download forecast CSVs

Tooltips:

Integrated help and metric descriptions

7. Conclusion & Recommendations
Best Performing Model: LSTM (R² ≈ 0.965, MAE ≈ 0.066)

Recommended Setup: Use LSTM + XGBoost ensemble for robust accuracy and interpretability.

8. Future Work
Incorporate additional environmental and socioeconomic variables.

Explore Transformer-based models.

Implement real-time pipelines for adaptive forecasting.

Analyze spatial–temporal patterns across cities.
