---
layout: page
title: NASA Airport Throughput Prediction Challenge
description: Developed a deep learning model to forecast airport throughput for U.S. airports
img: assets/img/plane.jpg
importance: 1
category: project
related_publications: False
---

# Project Summary: NASA Airport Throughput Prediction Challenge

## 1. Project Overview
In the fall of 2024, my lab participated in the [NASA Airport Throughput Prediction Challenge](https://bitgrit.net/competition/23). The challenge, part of the NASA Air Traffic Management - eXploration (ATM-X) project, focused on improving the operational efficiency of the National Airspace System (NAS). The core objective was to develop a high-precision regression model to forecast arrival throughput—the number of aircraft landing at an airport—over 15-minute intervals for a rolling 3-hour window.

By predicting congestion levels and runway configurations 180 minutes in advance, aviation operators can perform better flight planning, reduce fuel consumption, and mitigate the ripple effects of delays caused by convective weather or shifting airport configurations.

---

## 2. The Data Challenge
The project involved a multi-modal dataset exceeding **200 GB**, requiring sophisticated ingestion and feature engineering strategies to handle the sheer scale and variety of information.

### Data Domains:
* FUSER (Flight & Airport Data): Included D-ATIS configurations, actual arrival/departure detections, and FAA SWIM feeds. This provided the "ground truth" for target variables.
* Aviation Weather (METAR & TAF): Real-time hourly observations (METAR) and 24–30 hour terminal forecasts (TAF) containing wind speed, direction, visibility, and cloud ceiling data.
* CWAM (Convective Weather Avoidance Model): Complex HDF5 files containing spatial polygons representing areas with high probabilities of convective weather impact at various flight levels.
* LAMP: Short-term localized MOS forecasts used to bridge the gap between hourly observations and provide more granular predictions.



---

## 3. Distributed Data Pipeline & Orchestration
To handle the scale of the 12-month cyclical data split (24 days training / 8 days testing), we built a robust, automated pipeline using enterprise-grade distributed tools.

### Orchestration with Apache Airflow
We utilized Apache Airflow to manage the project's Directed Acyclic Graphs. Airflow acted as the centralized control plane, ensuring tasks were executed in the correct sequence:
* Cyclical Scheduling: Automated the 24/8 day split logic across a full year of data.
* Data Integrity: Implemented sensors to ensure upstream weather files (TAF/METAR) were fully downloaded and validated before triggering downstream Spark jobs.
* Fault Tolerance: Managed the long-running extraction of `.bz2` and `.h5` files, providing automatic retries for multi-hour processing tasks.

### Big Data Processing with Apache Spark
With 200 GB of data, traditional Pandas-based processing was unfeasible. Apache Spark enabled distributed feature engineering across a cluster:
* Distributed Spatial Joins: We utilized PySpark in conjunction with spatial libraries (like Shapely) to intersect flight paths with CWAM polygons. This allowed us to calculate weather impacts across thousands of coordinates in parallel.
* Point-in-Time Joins: To prevent data leakage, Spark was used to strictly join only the weather forecasts available at or before the specific prediction timestamp $T$.
* Feature Scaling: Spark window functions were used to compute rolling historical averages and airport-specific congestion trends across 53 major U.S. airports.



---

## 4. Modeling & Architecture
The forecasting engine utilized a hybrid architecture designed to capture both tabular relationships (weather/IDs) and temporal sequences (traffic flow).

### Model 1: LightGBM (Gradient Boosting)
LightGBM served as the tabular specialist, processing airport configurations and weather indices.
* Pros: Ultra-scalable via Gradient-based One-Sided Sampling; handles high-cardinality categorical data (53 airport IDs) efficiently; captures sharp non-linear threshold effects in weather data.
* Cons: No inherent temporal awareness; requires manual "lag" feature engineering to see past trends.



### Model 2: LSTM (Long Short-Term Memory)
The LSTM network was tasked with the memory of the system, processing sequences of arrival buckets to detect momentum.
* Pros: Native awareness of time-series dependencies; automatically learns temporal relationships through its Forget Gate architecture.
* Cons: Computationally expensive to train on 200 GB of data;



### Hybrid Ensemble Logic
Our 10th-place solution used Ensemble Stacking. Predictions from the LSTM were fed as meta-features into the LightGBM model. This combined the temporal insights of deep learning with the robust categorical handling of gradient boosting.

---

## 5. Results & Impact
* Metric: Evaluated via a normalized Root Mean Squared Error (RMSE) score: $Score = \exp(-RMSE/10)$.
* Performance: Achieved a final predictive accuracy of **78.77%**.
* Standing: Ranked **10th out of 53 teams** nationally.
* Code: Unfortunetly, our lab didn't make the code public, but you can see our standings on the leaderboard [here](https://bitgrit.net/competition/23)