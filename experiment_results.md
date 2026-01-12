```
# Deliverable 1: Experiment Results

## Executive Summary

**Objective:** Determine how much the rent prediction model can be improved using census data and other techniques.

**Key Finding:** Census data does NOT improve model performance. However, removing outliers from training data improves performance by 7.2%.

**Best Model:** Baseline features + outlier removal → 5.44% MAPE (vs. 5.86% baseline)

---

## Experiments Conducted

### Part 1: Census Data Evaluation

**Tool Used:** MLflow
**Metric:** Median Absolute Percentage Error (MAPE)

#### 1. Baseline Model (No Census Data)
**Features:** latitude, longitude, property_type, sqft, beds, baths, days_since_2014

**Results:**
- Train MAPE: 3.50%
- Test MAPE: 5.86%
- Overfitting Gap: 2.35%

---

#### 2. With Raw Census Features
**Features:** Baseline + population, housing, education, transportation

**Results:**
- Train MAPE: 3.57%
- Test MAPE: 5.98%
- Performance vs Baseline: -2.1% (WORSE)

---

#### 3. With Engineered Census Features
**Features:** Baseline + education_rate, housing_density, commuter_rate, population_log

**Results:**
- Train MAPE: 3.58%
- Test MAPE: 6.04%
- Performance vs Baseline: -3.1% (WORSE)

---

#### 4. Census Features Only
**Features:** education_rate, housing_density, commuter_rate, population_log

**Results:**
- Train MAPE: 13.14%
- Test MAPE: 14.27%
- Performance vs Baseline: -143.5% (MUCH WORSE)

---

### Part 2: Model Improvements (Alternative Approaches)

Since census data didn't help, we tested other improvement strategies:

#### 5. Remove Training Outliers
**Method:** Remove prices beyond 3 standard deviations from mean

**Results:**
- Removed 849 outliers (1.8% of training data)
- Test MAPE: 5.44%
- Improvement: +0.42pp (7.2% relative improvement)

---

#### 6. Simplified Transformation
**Method:** Log transformation only (no z-score normalization)

**Results:**
- Test MAPE: 5.77%
- Improvement: +0.09pp (1.5% relative improvement)

---

## Final Comparison

| Approach | Test MAPE | Change vs Baseline |
|----------|-----------|-------------------|
| Original Baseline | 5.86% | — |
| Census Data (best) | 5.98% | -0.12pp (worse) |
| Remove Outliers | 5.44% | +0.42pp (BEST) |
| Log-only Transform | 5.77% | +0.09pp |
| Feature Engineering | 6.27% | -0.41pp (worse) |
| Hyperparameter Tuning | 5.96% | -0.10pp (worse) |

---

## Conclusions

### 1. Census Data Analysis
**Finding:** Census data does NOT add predictive value and actually degrades performance.

**Reasons:**
- Block group-level census data is too coarse-grained
- Property-level features (sqft, beds, location) already capture pricing signal
- Census demographics likely correlated with location coordinates already in model

**Recommendation:** EXCLUDE census features from production model.

---

### 2. Best Model Improvement
**Finding:** Removing outliers from training data improves MAPE from 5.86% to 5.44% (7.2% improvement).

**Why it works:**
- Extreme prices (very cheap or luxury properties) introduce noise
- Model generalizes better when trained on typical market conditions
- 3 sigma filter removes only 1.8% of data but improves robustness

**Recommendation:** INCLUDE outlier filtering in production pipeline.

---

### 3. Additional Observations
- Simplified log transformation (5.77% MAPE) performs nearly as well as z-normalized approach
- Feature engineering (sqft per bed/bath) did not help - likely redundant with existing features
- Hyperparameter tuning provided minimal gains - default parameters are well-suited

---

## Production Recommendation

**Winning Configuration:**

Step 1: Filter outliers from training data
price_mean = train_df['price'].mean()
price_std = train_df['price'].std()
train_clean = train_df[
    (train_df['price'] > price_mean - 3*price_std) & 
    (train_df['price'] < price_mean + 3*price_std)
]

Step 2: Use baseline features only
features = ['latitude', 'longitude', 'property_type', 'sqft', 
            'beds', 'baths', 'days_since_2014']

Step 3: Standard XGBoost with default hyperparameters
model = xgb.XGBRegressor(
    objective="reg:squarederror",
    random_state=111,
    n_estimators=100,
    max_depth=6,
    learning_rate=0.3
)

**Expected Performance:** 5.44% MAPE (7.2% better than original proof-of-concept)

---

## MLflow Experiment Tracking

All experiments logged to MLflow with:
- Parameters: feature lists, model hyperparameters, data preprocessing
- Metrics: train_mape, test_mape, overfitting_gap
- Models: Serialized XGBoost models for reproducibility

Access logs via: mlflow ui in the project directory
```

---