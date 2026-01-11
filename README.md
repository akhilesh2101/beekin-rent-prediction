# Beekin Rent Prediction Model - Take-Home Assignment
Candidate:Akhilesh Sureddi 

## Overview
This submission contains four deliverables for improving the Beekin rent prediction model:
1. Census data experimentation with MLflow tracking
2. Strategic memo on model risks and design
3. Production model refresh pipeline
4. Performance monitoring system

## Structure
- `01_experiments/` - Census data impact analysis (Deliverable 1)
- `02_memo/` - Strategic planning memo (Deliverable 2)
- `03_production_refresh/` - Model refresh automation (Deliverable 3)
- `04_monitoring/` - Performance monitoring (Deliverable 4)

## Setup Instructions

### Requirements
```bash
pip install -r requirements.txt
```

### Data Files Expected
Place the following files in a `data/` directory:
- `train.csv`
- `test.csv`
- `census.csv`
- `data_2022.csv`

### Running the Code

**Deliverable 1 - Experiments:**
```bash
cd 01_experiments
python experiment_tracking.py
mlflow ui  # View results at http://localhost:5000
```

**Deliverable 3 - Model Refresh:**
```bash
cd 03_production_refresh
python model_refresh.py
```

**Deliverable 4 - Monitoring:**
```bash
cd 04_monitoring
python monitoring_function.py
```

## Key Findings
- Census-derived features improved test MAPE from 5.86% to ~5.3% (9% improvement)
- Recommended weekly refresh cadence with monotonicity constraints
- Monitoring thresholds: >1pp MAPE increase triggers rollback

## Questions?
Contact: akhileshsureddi@gmail.com
