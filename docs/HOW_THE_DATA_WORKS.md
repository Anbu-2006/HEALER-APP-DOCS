# Healer App — How the Data Works

This document explains the data pipeline of the Healer App machine learning system, outlining the feature engineering process, sensor conversions, and dataset composition.

---

## 📈 Feature Engineering Pipeline

Mobile sensors record data at high frequencies (50Hz - 100Hz). Feeding raw streams directly into neural networks on mobile devices is computationally prohibitive. Instead, Healer uses a **statistical feature extractor** that transforms raw time-series data into a single summary vector representing a sliding time window.

```
[Raw Sensors 50Hz] ──> [10s Sliding Window] ──> [Compute Stats (Mean, RMS, ZCR...)] ──> [48-Feature Vector]
```

### 1. Sliding Window & Stride
* **Window Size:** 10 samples (representing a 10-second window at 1Hz, matching the app's real-time aggregation thread).
* **Stride:** 5 samples (50% overlap). Used during preprocessing to augment training data for robust boundary detection.

### 2. Standardized Unit Conversions
To ensure the training data matches real-time mobile hardware, the preprocessing pipeline applies strict unit scaling:
* **Accelerometer:** Normalized to $m/s^2$ (gravity constant $1g = 9.81 m/s^2$). Raw datasets storing values in $g$-units are automatically scaled.
* **Velocity:** Standardized to meters per second ($m/s$). Speed datasets in kilometers per hour ($km/h$) are converted using the formula: $v_{mps} = v_{kmh} / 3.6$.

---

## 📊 The 48-Feature Vector

Each preprocessed data window is mapped to a flat array of **48 floating-point features** that are fed directly into the model inputs:

### Statistical Features (Indices 0–35)
For each of the six sensor axes (Accelerometer X/Y/Z, Gyroscope X/Y/Z), the app computes six time-domain statistics:
1. **Mean:** Central tendency of the signal.
2. **Standard Deviation (Std):** Variation/spread of the sensor forces.
3. **Minimum:** Lowest registered force vector.
4. **Maximum:** Peak impact force vector.
5. **Root Mean Square (RMS):** Measure of the signal's total power.
6. **Zero-Crossing Rate (ZCR):** Number of times the signal crosses its mean value (helps differentiate sudden bumps from steady vibration).

### Contextual Features (Indices 36–47)
These scalar parameters integrate environment and device state details:

| Index | Feature | Purpose / Definition |
| :--- | :--- | :--- |
| **36** | `gForce_mean` | Average total G-force magnitude. |
| **37** | `gForce_max` | Maximum total G-force magnitude (main crash indicator). |
| **38** | `dbfs_mean` | Average ambient sound amplitude in decibels (dBFS). |
| **39** | `dbfs_max` | Peak sound amplitude registered during window. |
| **40** | `speed_mean` | Average vehicle speed in meters per second ($m/s$). |
| **41** | `proximity_ratio` | Percentage of window where proximity was "NEAR". |
| **42** | `lux_mean` | Average ambient light level (helps identify glovebox drops). |
| **43** | `pressure_delta` | Barometer pressure differential (airbag deployment signature). |
| **44** | `activity_encoded`| Current user activity (3.0 = `IN_VEHICLE`). |
| **45** | `audio_crash_mean`| Running average of frequency-domain crash scores. |
| **46** | `audio_crash_max` | Peak frequency-domain crash score. |
| **47** | `audio_dom_band`  | Dominant acoustic band (engine hum vs. metal/glass clashing). |

---

## 🗄️ Dataset Composition

The model was trained on a combination of real-world and synthetic datasets to cover both normal driving habits and crash telemetry:

1. **Kaggle Accident Telemetry Dataset:**
   Contains real smartphone sensor streams from road accidents, recording severe longitudinal/lateral crash spikes (label `1`) and normal travel (label `0`).
2. **Mendeley Driving Behavior Dataset:**
   Contains high-resolution sensor recordings from drivers executing aggressive maneuvers (lane changes, U-turns, sudden braking, and sudden acceleration). All records are labeled safe (`0`) to prevent false triggers during sharp maneuvers.
3. **Synthetic Edge Case Augmentation:**
   Mathematically simulated extreme drops, low-speed impacts, and complex rollover dynamics to balance classes and establish clear threshold boundaries.
