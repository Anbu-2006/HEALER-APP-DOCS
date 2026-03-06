# 🎯 Healer App — Detection System Completeness Checklist

**Last Updated:** March 2026  
**Detection Layer Status:** 100% Complete (Rule-Based + Frequency Analysis)  
**ML Layer Status:** 20% Complete (Scaffolding + Feature Extraction)

---

## 1. Sensor Coverage Audit

Every mobile-accessible hardware sensor relevant to accident detection has been evaluated and integrated where applicable.

### ✅ Implemented Sensors (10 Active)

| # | Sensor | Android Type | Provider File | Purpose | Status |
|---|--------|-------------|---------------|---------|--------|
| 1 | **Accelerometer** | `TYPE_LINEAR_ACCELERATION` / `TYPE_ACCELEROMETER` + HighPassFilter | `AccelProvider.kt` | Impact G-force, jerk, axis classification | ✅ Active |
| 2 | **Gyroscope** | `TYPE_GYROSCOPE` | `GyroProvider.kt` | Sudden rotation (spin, flip) | ✅ Active |
| 3 | **Gravity** | `TYPE_GRAVITY` | `GravityProvider.kt` | Orientation tracking, rollover detection | ✅ Active |
| 4 | **Microphone** | `AudioRecord` (44.1 kHz PCM) | `SoundProvider.kt` | Crash/impact sound (dBFS level) | ✅ Active |
| 5 | **Light** | `TYPE_LIGHT` | `LightProvider.kt` | Sudden darkness (airbag, phone flung) | ✅ Active |
| 6 | **Proximity** | `TYPE_PROXIMITY` | `ProximityProvider.kt` | In-pocket detection (confidence boost) | ✅ Active |
| 7 | **Barometer** | `TYPE_PRESSURE` | `BarometerProvider.kt` | Cabin pressure spike on impact | ✅ Active (optional HW) |
| 8 | **GPS / Network** | `LocationManager` | `LocationProvider.kt` | Speed for gating + emergency coordinates | ✅ Active |
| 9 | **Rotation Vector** | `TYPE_ROTATION_VECTOR` | `OrientationHelper.kt` | Device→World frame rotation matrix | ✅ Active |
| 10 | **Step Counter** | `TYPE_STEP_COUNTER` + GPS speed | `ActivityDetector.kt` | STILL/WALKING/IN_VEHICLE classification | ✅ Active |

### ⏭️ Sensors Evaluated & Deferred (Not Critical)

| Sensor | Android Type | Why Deferred | Priority |
|--------|-------------|--------------|----------|
| **Magnetometer** | `TYPE_MAGNETIC_FIELD` | Compass heading; not needed — RotationVector already fuses magnetometer internally. Directional classification handled by accelerometer axis analysis. | Low — Phase 2 |
| **Temperature** | `TYPE_AMBIENT_TEMPERATURE` | Environmental context only; extremely rare on consumer devices (<5% availability). No impact on accident detection accuracy. | None — Skip |
| **Humidity** | `TYPE_RELATIVE_HUMIDITY` | Weather context; no direct accident correlation. Rare hardware. | None — Skip |
| **Heart Rate** | `TYPE_HEART_RATE` | Only on wearables; not available on phones. Would be useful for fall detection on Wear OS. | None — Phone scope |
| **Significant Motion** | `TYPE_SIGNIFICANT_MOTION` | One-shot trigger; too coarse for accident classification (triggers on any movement). Step Counter + Accel already covers this. | None — Redundant |

### 🔍 Verdict: **All practical mobile sensors for accident detection are integrated.**

> The Rotation Vector sensor (`TYPE_ROTATION_VECTOR`) internally fuses the accelerometer, gyroscope, AND magnetometer — so magnetometer data is already utilized for world-frame normalization without needing a separate `MagnetometerProvider`.

---

## 2. Detection Algorithm Verification

### 2.1 Fusion Logic — Impact-Anchored Event Correlation ✅

| Channel | Source Provider | Trigger Condition | Timestamp Field |
|---------|----------------|-------------------|-----------------|
| **Shake/Impact** | AccelProvider → ShakeDetector | G-force > threshold (4g+ speed-adjusted) AND sustained 3+ samples/200ms OR jerk ≥ 8.0 g/s OR buffer spike (max-mean > 3.0g) | `shakeDetectedTime` |
| **Sound** | SoundProvider | dBFS > -50.0 (GATED — only fires above threshold) | `soundDetectedTime` |
| **Gyro Spin** | GyroProvider | Rotation magnitude ≥ 300 °/s | `gyroDetectedTime` |
| **Sudden Darkness** | LightProvider | Lux < 30% of baseline AND >50% drop | `lightDarknessTime` |
| **Rollover** | GravityProvider | Any axis shift > 7.0 m/s² from baseline | `rolloverTime` |
| **Pressure Spike** | BarometerProvider | Δ ≥ 0.5 hPa within 100ms | `barometerSpikeTime` |

**Fusion Rules:**
1. Impact (shake) MUST be present — mandatory anchor
2. ≥1 corroborating signal within **1500ms** fusion window
3. Probability score must be ≥ **70%** after activity multiplier
4. NOT in WALKING state (activity suppression)
All rules must pass → ACCIDENT CONFIRMED

### 2.2 Probability Scoring — Weighted Confidence ✅

| Component | Points | Condition |
|-----------|--------|-----------|
| Impact detected | +30 | Any impact type classified |
| G-force spike | +25 | > 2.5g |
| Sound exceeded | +15 | dBFS > -50.0 |
| Secondary motion | +5 | Post-impact motion detected within 2000ms |
| Proximity: NEAR | +15 | Phone in pocket/covered |
| Light: very dark | +10 | Lux < 20% of baseline |
| Light: moderate drop | +5 | Lux < 40% of baseline |
| Gravity: upside down | +20 | Device inverted |
| Gravity: orientation changed | +10 | Axis deviation from baseline |
| Barometer spike | +20 | Pressure spike within last 3000ms |
| **Max possible base** | **155** | (before multiplier) |

| Activity State | Multiplier | Rationale |
|----------------|-----------|-----------|
| IN_VEHICLE | ×1.5 | High confidence — likely real accident |
| STILL | ×1.0 | Normal baseline |
| UNKNOWN | ×1.0 | Default fallback |
| WALKING | ×0.3 | Strong suppression — likely phone drop, stumble |

**Final Score** = clamp((base_sum) × multiplier, 0, 100)

### 2.3 False-Positive Suppression Rules ✅

| Rule | Implementation | Status |
|------|---------------|--------|
| **Walking Suppression** | `ActivityDetector.isLikelyWalking()` → skip accident confirmation entirely | ✅ Active |
| **Walking Multiplier** | 0.3× on probability score when WALKING detected | ✅ Active |
| **Speed Gating** | Thresholds harder to trigger at <3 km/h (×1.3), easier at >50 km/h (×0.7) | ✅ Active |
| **Alert Debounce** | 5000ms minimum between consecutive accident alerts | ✅ Active |
| **Shake Debounce** | 800ms minimum between consecutive impact detections | ✅ Active |
| **Sound Debounce** | 250ms minimum between sound event updates | ✅ Active |
| **Barometer Debounce** | 2000ms between pressure spike alerts | ✅ Active |
| **Pair Requirement** | Single-sensor events alone NEVER trigger accident — impact is mandatory anchor + corroboration | ✅ Active |
| **Probability Gate** | Score must be ≥70% after activity multiplier to confirm accident | ✅ Active |
| **Sustained Impact** | 3+ qualifying samples within 200ms required (filters single-spike drops) | ✅ Active |
| **Sound Gating** | Sound event only set when dBFS > -50 (not every sample) | ✅ Active |
| **Post-Impact Validation** | 2000ms window monitors for secondary motion: \|gForce-1.0\| > 0.5 | ✅ Active |
| **Oldest Event Expiry** | Stale timestamps beyond fusion window are not considered | ✅ Active |

### 2.4 Impact Classification — Axis-Based ✅

| Impact Type | Dominant Axis | Threshold (g) | Speed-Adjusted Range |
|-------------|---------------|---------------|---------------------|
| **Vertical Impact** | Z-axis | 4.0g | 2.80g – 5.20g |
| **Lateral Impact** | Y-axis | 3.0g | 2.10g – 3.90g |
| **Forward/Rear Impact** | X-axis | 2.0g | 1.40g – 2.60g |
| **Rollover** | Gravity shift | 7.0 m/s² | Fixed (no speed adjustment) |
| **General Impact** | Jerk-based | 8.0 g/s | Fixed |
| **Buffer Spike** | Pattern-based | (max-mean) > 3.0g | Fixed |
| **Sustained** | Sample count | 3+ in 200ms | Fixed |

---

## 3. Threshold Calibration Strategy

### 3.1 Current Approach: Crash-Realistic Thresholds ✅

All thresholds are tuned to crash-realistic levels (well above human movement range):
- G-force thresholds: Frontal 4.0g, Side 3.0g, Rollover 2.0g
- Sound threshold: -50.0 dBFS (gated — only fires event above threshold)
- Gyro peak: 300 °/s
- Barometer spike: 0.5 hPa / 100ms
- Gravity rollover: 7.0 m/s² axis shift
- Light darkness: <30% of calibrated baseline

### 3.2 Speed-Gating Profiles ✅

| Speed Range | Multiplier | Effect |
|-------------|-----------|--------|
| < 3 km/h (stationary/walking) | ×1.3 | Harder to trigger — reduces false positives from drops |
| 3–20 km/h (urban driving) | ×1.0 | Normal sensitivity |
| 20–50 km/h (city/suburban) | ×0.85 | Slightly more sensitive |
| > 50 km/h (highway) | ×0.7 | Most sensitive — high-speed impacts are severe |

### 3.3 Auto-Calibration (Per-Session) ✅

| Sensor | Calibration Method | Window |
|--------|-------------------|--------|
| Sound | 1-second auto-baseline at start; capped at -3.0 dBFS | First 1000ms |
| Light | 10-sample baseline averaging | First 10 readings |
| Gravity | 20-sample baseline for orientation reference | First 20 readings |

### 3.4 Device-Profile Calibration ✅ (NEW)

Different devices have different sensor sensitivities (±20%). The system now includes:
- Auto-detect device model via `Build.MANUFACTURER` + `Build.MODEL`
- 30+ device profiles in `DeviceProfileRegistry.kt` (Samsung, Pixel, OnePlus, Xiaomi, Redmi, POCO, Realme, Oppo, Vivo, Motorola, Nothing, Huawei, Honor, Sony, Nokia)
- Per-device multipliers for accelerometer, gyroscope, sound, light, barometer
- Generic fallback (×1.0) if device unknown
- Profile logged to CSV for ML training normalization

See: `docs/THRESHOLD_CALIBRATION_REPORT.md` for detailed per-threshold justification.

---

## 4. Edge Cases Handled

| Edge Case | How It's Handled | Status |
|-----------|-----------------|--------|
| **No gyroscope** | `hasSensor()` check → skip, fusion continues with remaining channels | ✅ |
| **No barometer** | `hasSensor()` check → skip, pressure spike channel inactive | ✅ |
| **No GPS permission** | Location optional; speed defaults to 0 m/s; speed-gating uses ×1.0 | ✅ |
| **No microphone permission** | RECORD_AUDIO is required; app blocks start without it | ✅ |
| **No step counter** | ActivityDetector falls back to speed-only classification | ✅ |
| **No linear acceleration sensor** | AccelProvider uses raw TYPE_ACCELEROMETER + HighPassFilter fallback | ✅ |
| **No rotation vector** | OrientationHelper unavailable; accel processed in device frame (degraded accuracy) | ✅ |
| **Phone in pocket (proximity NEAR)** | Boost confidence +15 (real accidents = phone stays in pocket) | ✅ |
| **Phone exposed (proximity FAR)** | No boost; could be phone drop | ✅ |
| **User walking + phone drop** | Walking ×0.3 multiplier + walking suppression → no false alert | ✅ |
| **Rough road vibrations** | Pair requirement (2+ channels) prevents single-accel false positives | ✅ |
| **Loud music in car** | Sound calibration establishes high baseline; threshold relative to norm | ✅ |
| **Driving over speed bumps** | Speed-gated thresholds + activity multiplier mitigate | ✅ |
| **Phone thrown on table** | Short impact; no sound/light/proximity correlation → no pair → no alert | ✅ |
| **Screen off / app backgrounded** | Currently requires foreground; background service planned | ⏳ Planned |

---

## 5. Acoustic Analysis — Current Implementation

### dBFS Level Detection ✅
- **Metric:** dBFS (decibel full-scale) — overall loudness
- **Method:** RMS amplitude of PCM samples → dBFS conversion
- **Threshold:** > -50.0 dBFS triggers sound event
- **Calibration:** 1-second baseline at start
- **Polling:** 200ms intervals

### FFT Frequency-Band Analysis ✅ (NEW)
- **Processor:** `AudioSignalProcessor.kt` in `ml/` package
- **Algorithm:** Cooley-Tukey radix-2 in-place FFT
- **Input:** Raw PCM 16-bit audio buffer (44.1 kHz)
- **Band Classification:**
  - 0–500 Hz: Engine rumble, road noise (weight: 0.2×)
  - 500 Hz–2 kHz: Impact/collision (weight: 0.5×)
  - 2–6 kHz: Glass breaking, metal impact, airbag (weight: 1.0×) ← **Highest confidence**
  - 6–10 kHz: Tire screech (weight: 0.3×)
- **Output:** `AudioSignature` with band energies, crash score (0–1), spectral centroid, spectral entropy
- **Integration:** `SoundProvider` runs FFT every 200ms, `DetectionCoordinator` boosts confidence +5/+10 if crash signature detected
- **Logging:** `TrainingLogger` exports `audio_crash_score` and `audio_dominant_band` columns for ML training

> **Verdict:** Both dBFS level AND frequency-band analysis are now active. The FFT crash signature score provides additional confidence when 2–6 kHz bands dominate (glass/metal/airbag sounds).

---

## 6. Sensor Fusion Completeness Matrix

This matrix shows which sensor pairs provide meaningful accident signals when correlated:

| | Impact | Sound | Gyro | Darkness | Rollover | Baro Spike |
|---|:---:|:---:|:---:|:---:|:---:|:---:|
| **Impact** | — | 🔴 HIGH | 🔴 HIGH | 🟡 MED | 🔴 HIGH | 🔴 HIGH |
| **Sound** | 🔴 HIGH | — | 🟡 MED | 🟡 MED | 🟡 MED | 🟡 MED |
| **Gyro** | 🔴 HIGH | 🟡 MED | — | 🟡 MED | 🔴 HIGH | 🟡 MED |
| **Darkness** | 🟡 MED | 🟡 MED | 🟡 MED | — | 🟡 MED | 🟡 MED |
| **Rollover** | 🔴 HIGH | 🟡 MED | 🔴 HIGH | 🟡 MED | — | 🟡 MED |
| **Baro Spike** | 🔴 HIGH | 🟡 MED | 🟡 MED | 🟡 MED | 🟡 MED | — |

🔴 **HIGH** = Strong accident indicator when paired  
🟡 **MED** = Supportive signal, adds confidence

**All 15 possible pairs are actively monitored** by `hasPairWithinWindow()`.

---

## 7. Detection System Maturity Summary

| Area | Status | Notes |
|------|--------|-------|
| Sensor coverage (hardware) | ✅ 100% | All practical mobile sensors integrated |
| Sensor abstraction (BaseSensor) | ✅ 100% | Uniform lifecycle management |
| Impact classification (axis-based) | ✅ 100% | Frontal, Lateral, Vertical, Rollover |
| Multi-sensor fusion (6 channels) | ✅ 100% | 1500ms time-window correlation |
| Probability scoring (weighted) | ✅ 100% | 7 signal components + activity multiplier |
| False-positive suppression | ✅ 100% | Walking, debounce, pair requirement, speed-gating |
| Auto-calibration (per-session) | ✅ 100% | Sound, Light, Gravity |
| World-frame normalization | ✅ 100% | OrientationHelper rotation matrix |
| Speed-gated thresholds | ✅ 100% | 4 speed bands with multipliers |
| Circular buffer (ML-ready) | ✅ 100% | 100 samples / 2 seconds, thread-safe |
| Training data export | ✅ 100% | CSV with world-frame + activity state |
| Edge case handling | ✅ 95% | Graceful degradation for missing hardware |
| Acoustic frequency analysis | ✅ 100% | FFT 4-band analysis via AudioSignalProcessor |
| Device-specific calibration | ✅ 100% | DeviceProfileRegistry (30+ device models) |
| ML model inference | 📋 0% | Phase 2 — scaffolding ready |
| Background detection | 📋 0% | Requires foreground service |

### Overall Detection Level: **100% Complete (Rule-Based)**

> All detection algorithms, sensors, frequency analysis, and device calibration are fully implemented. Remaining work (ML inference, background service) is infrastructure — not detection logic.

---

## 8. What's Next for Detection (Roadmap)

| Phase | Milestone | Timeline | Impact |
|-------|-----------|----------|--------|
| **Phase 1** ✅ | Rule-based fusion | Complete | Functional accident detection |
| **Phase 1** ✅ | Acoustic frequency bands (FFT) | Complete | Better crash vs noise distinction |
| **Phase 1** ✅ | Device-profile calibration | Complete | Tighter thresholds per device |
| **Phase 2** 📋 | Background service | Q3 2026 | 24/7 detection |
| **Phase 2** 📋 | Emergency notification | Q3 2026 | Auto SOS + contacts |
| **Phase 3** 📋 | TFLite model integration | Q4 2026 | ML-based classification |
| **Phase 3** 📋 | Cloud model updates | Q1 2027 | OTA model refinement |

---

**Last Updated:** March 2026  
**Author:** Anbu (@Anbu-2006)

