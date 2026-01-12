# Beekin Data Scientist Take-Home Assignment

**Candidate:** Akhilesh Sureddi


---

## Overview

This repository contains my submission for the Beekin Data Scientist position. The objective was to improve a rent prediction model for Idaho properties beyond the initial XGBoost proof-of-concept.

**Key Achievement:** Improved model performance from 5.86% MAPE to 5.44% MAPE (7.2% relative improvement) through outlier filtering.

---

## Deliverables

### Deliverable 1: Experiment Tracking & Model Improvement

**Location:** deliverable_1_experiments/

**Objective:** Determine if census data and other improvements enhance model performance.

**Experiments Conducted:**
1. Census data integration (raw and engineered features)
2. Outlier removal from training data
3. Alternative target transformations
4. Feature engineering
5. Hyperparameter tuning

**Key Findings:**
- Census data DOES NOT improve performance (degrades to 5.98-6.04% MAPE)
- Removing outliers (3 sigma filter) IMPROVES performance to 5.44% MAPE (+7.2%)
- Simplified log transformation slightly improves to 5.77% MAPE

**Recommendation:** Use baseline features + outlier filtering for production.

**Results Summary:**

| Approach | Test MAPE | vs Baseline |
|----------|-----------|-------------|
| Original Baseline | 5.86% | — |
| Remove Outliers | 5.44% | +0.42pp |
| Log-only Transform | 5.77% | +0.09pp |
| Census Data | 5.98% | -0.12pp |

**Files:**
- experiment_tracking.py - Complete MLflow experiment code
- experiment_results.md - Detailed analysis and findings

---

### Deliverable 2: Model Risk & Design Memo

**Location:** deliverable_2_memo/model_risk_design_memo.pdf

**Topics Covered:**
1. Feature Risks: Exclude race/ethnicity, school districts for legal compliance
2. Temporal Design: Weekly refresh with exponential time weighting
3. Failure Detection: Data drift, prediction distribution, staleness monitoring
4. Monotonicity Trade-offs: Enforce on sqft; accept 0.5% accuracy loss for interpretability

---

### Deliverable 3: Production Refresh Pipeline

**Location:** deliverable_3_production/

**Objective:** Automate model training when new data arrives.

**Features:**
- Outlier filtering (3 sigma on training data)
- Feature engineering pipeline
- Model versioning with metadata
- Timestamp-based storage

**Files:**
- model_refresh.py - Production-ready refresh script
- architecture_diagram.png - End-to-end ML pipeline

**Pipeline:** Data Ingestion → Validation → Training → Model Registry → Monitoring → Deployment

---

### Deliverable 4: Performance Monitoring

**Location:** deliverable_4_monitoring/model_monitoring.py

**Objective:** Detect model degradation across refreshes.

**Thresholds:**
- PASS: MAPE change ≤ 1.0pp AND ≤ 15% relative
- WARNING: MAPE increase > 15% relative
- FAIL: MAPE increase > 1.0pp → Rollback

**Function:** monitor_model_performance() evaluates current vs. previous model and returns deployment decision.

---

## Setup Instructions
```bash
# Install dependencies
pip install -r requirements.txt

# Run experiments (Deliverable 1)
cd deliverable_1_experiments
python experiment_tracking.py

# Run model refresh (Deliverable 3)
cd deliverable_3_production
python model_refresh.py

# Run monitoring (Deliverable 4)
cd deliverable_4_monitoring
python model_monitoring.py
```

---

## Key Results

### Model Performance Progression

**Proof-of-Concept (provided):** 5.78% MAPE
**Our Baseline (replicated):** 5.86% MAPE
**Final Improved Model:** 5.44% MAPE

**Improvement:** 0.42 percentage points (7.2% relative improvement)

### What Worked
- Outlier removal (3 sigma filter on training prices)
- Simplified log transformation
- Baseline features (location, property characteristics, time)

### What Didn't Work
- Census data (population, education, housing statistics)
- Feature engineering (sqft per bed/bath ratios)
- Aggressive hyperparameter tuning

---

## Production Recommendation

**Winning Configuration:**

Filter training outliers:
price_mean, price_std = train_df['price'].mean(), train_df['price'].std()
train_clean = train_df[(train_df['price'] > price_mean - 3*price_std) & 
                       (train_df['price'] < price_mean + 3*price_std)]

Features: latitude, longitude, property_type, sqft, beds, baths, days_since_2014

Model: XGBRegressor(objective="reg:squarederror", random_state=111,
                    n_estimators=100, max_depth=6, learning_rate=0.3)

**Expected Performance:** 5.44% Median Absolute Percentage Error

---

## Technologies Used

- Python 3.x
- XGBoost - Gradient boosting
- MLflow - Experiment tracking
- Pandas/NumPy - Data processing
- scikit-learn - Evaluation metrics

---

## Contact

For questions: akhileshsureddi@gmail.com
