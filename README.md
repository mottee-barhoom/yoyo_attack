# 🛡️ Kubernetes YoYo Attack Detection - Base Dataset

> **A leakage-free, production-viable dataset for machine learning-based intrusion detection in Kubernetes environments**

---

## 📖 Overview & Significance

This dataset contains **2,515 labeled samples** collected from **48 controlled YoYo attack experiments** on a Kubernetes v1.27.3 cluster. It serves as the foundational input for developing ML-based detection systems that identify YoYo attacks targeting the Horizontal Pod Autoscaler (HPA).

### 🔍 Why This Dataset Matters
YoYo attacks exploit Kubernetes auto-scaling mechanisms by generating oscillating load patterns, causing:
- **Resource waste** from repeated scaling cycles
- **Service instability** and potential SLA violations  
- **Increased operational costs** (15-25% infrastructure overhead)

**Research Gap:** Most existing solutions suffer from information leakage, require intrusive application instrumentation, or use unrealistic experimental setups that prevent production deployment.

**Our Solution:** This dataset enforces **strict leakage prevention** by excluding 17 columns (metadata, orchestrator states, derived features), retaining only **34 genuine system-level metrics** measurable in live Kubernetes environments without code modifications.

---

## 📦 Repository Contents

| File / Directory | Description |
|------------------|-------------|
| `base_dataset.csv` | Clean dataset: **2,515 samples × 35 columns** (34 features + `label`) |
| `data_dictionary.csv` | Academic metadata: column names, types, missing counts, unique values |
| `dataset_stats.json` | Machine-readable summary statistics for automated pipelines |
| `analysis/` | Directory containing statistical reports and publication-ready visualizations |
| `scripts/` | Python scripts for dataset inspection and analysis |

---

## 📊 Dataset Statistics

| Metric | Value |
|--------|-------|
| **Total Samples** | 2,515 |
| **Base Features** | 34 system-level metrics |
| **Target Column** | `label` (`ATTACK` / `NORMAL`) |
| **Controlled Experiments** | 48 scenarios |
| **Sampling Interval** | 10 seconds |
| **Memory Footprint** | ~1.2 MB |

### Class Distribution
| Class | Samples | Percentage |
|-------|---------|------------|
| `ATTACK` | ~1,207 | 48.0% |
| `NORMAL` | ~1,308 | 52.0% |
| **Balance Ratio** | 0.92 | ✅ Well-balanced |

### Data Quality
- ✅ **Missing Values:** 0 (0.0%)
- ✅ **Duplicate Rows:** 0
- ✅ **Numeric Features:** 34
- ✅ **Categorical Features:** 0 (excluding label)

---

##  Column Filtering Strategy: 52 → 35

The raw experimental export contained **52 columns**. To ensure production viability and prevent data leakage, **17 columns were explicitly excluded** across three categories:

###  1. Metadata & Experiment Context (4 columns)
| Column | Reason for Exclusion |
|--------|---------------------|
| `experiment_id` | Trial identifier leaks experiment grouping |
| `timestamp` | Temporal anchor, not a measurable system metric |
| `attack_type` | Ground truth configuration causes direct leakage |
| `attack_power_K` | Attack intensity parameter, not system behavior |

### 🔒 2. Leakage-Prone HPA/Deployment State (5 columns)
| Column | Reason for Exclusion |
|--------|---------------------|
| `hpa_current_replicas` | Reveals active scaling state during attack window |
| `hpa_desired_replicas` | Future scaling target not observable at prediction time |
| `hpa_target_cpu_utilization` | Static configuration parameter |
| `hpa_current_cpu_utilization` | Directly correlated with scaling trigger |
| `deployment_available_replicas` | Orchestrator state reveals scaling intent |

###  3. Derived & Engineered Features (8 columns)
| Column | Reason for Exclusion |
|--------|---------------------|
| `label_numeric` | Redundant encoding of target label |
| `cpu_memory_ratio`, `cpu_per_pod`, `memory_per_pod` | Engineered ratio features (Phase 2) |
| `network_per_request`, `requests_per_pod` | Engineered ratio features (Phase 2) |
| `pod_oscillation`, `pod_change_rate` | Behavioral indicators (Phase 2) |
| `*_mean_3`, `*_std_3` | Temporal aggregations (Phase 2) |

**Result:** `52 - 17 = 35` columns retained (`34` genuine base metrics + `label`)

---

##  Visualizations

### Class Distribution
![Class Distribution](analysis/01_class_distribution.png)
*Figure 1: Balanced class distribution ensures robust model training without sampling bias.*

### Feature Type Distribution
![Feature Types](analysis/02_feature_types.png)
*Figure 2: All 34 base features are numeric system-level metrics suitable for ML algorithms.*

### Feature Correlation Heatmap
![Correlation Heatmap](analysis/04_correlation_heatmap.png)
*Figure 3: Correlation matrix reveals feature relationships and potential multicollinearity.*

### Top Feature Distributions
![Distribution 1](analysis/05_dist_1_*.png)
![Distribution 2](analysis/05_dist_2_*.png)
*Figure 4-9: Distribution plots for top 6 features by variance, split by class (`ATTACK` vs `NORMAL`).*

---

## 🔍 Feature Categories (34 Base Metrics)

The retained features span five measurement sources:

| Category | Count | Example Metrics |
|----------|-------|-----------------|
| **HTTP Metrics** | 8 | `http_request_rate`, `http_response_time_p90`, `http_error_rate` |
| **Network Metrics** | 8 | `network_in`, `network_out`, `network_errors_in`, `network_drop_out` |
| **Pod Metrics** | 8 | `pod_count`, `pod_ready_count`, `pod_cpu_usage_cores`, `pod_memory_usage_bytes` |
| **Resource Metrics** | 2 | CPU & Memory utilization cores/bytes |
| **HPA Metrics** | 8 | `hpa_efficiency`, `hpa_desired_replicas` (non-leaking variants) |

*See `data_dictionary.csv` for complete column descriptions, data types, and value ranges.*

---

## 🛠️ Usage Examples

### Python (Pandas)
```python
import pandas as pd

# Load base dataset
df = pd.read_csv("base_dataset.csv")

# Separate features and target
X = df.drop(columns=["label"])
y = df["label"]

print(f"Features: {X.shape[1]} columns")
print(f"Samples: {X.shape[0]} rows")
