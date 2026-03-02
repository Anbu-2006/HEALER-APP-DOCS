# 🎯 Healer App — Detection System Completeness Checklist

**Last Updated:** March 2026  
**Detection Layer Status:** 95% Complete (Rule-Based)  
**ML Layer Status:** 15% Complete (Scaffolding Only)

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

### 2.1 Fusion Logic — 6-Channel Event Correlation ✅

| Channel | Source Provider | Trigger Condition | Timestamp Field |
|---------|----------------|-------------------|-----------------|
| **Shake/Impact** | AccelProvider → ShakeDetector | G-force > threshold (speed-adjusted) OR jerk ≥ 2.0 g/s OR buffer spike | `shakeDetectedTime` |
| **Sound** | SoundProvider | dBFS > -50.0 (configurable) | `soundDetectedTime` |
| **Gyro Spin** | GyroProvider | Rotation magnitude ≥ 100 °/s | `gyroDetectedTime` |
| **Sudden Darkness** | LightProvider | Lux < 30% of baseline AND >50% drop | `lightDarknessTime` |
| **Rollover** | GravityProvider | Any axis shift > 7.0 m/s² from baseline | `rolloverTime` |
| **Pressure Spike** | BarometerProvider | Δ ≥ 0.5 hPa within 100ms | `barometerSpikeTime` |

**Fusion Rule:** Any 2 of the 6 channels triggering within **1500ms** = potential accident → proceed to probability scoring.

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
| **Pair Requirement** | Single-sensor events alone NEVER trigger accident — always requires 2+ correlated events | ✅ Active |
| **Post-Impact Validation** | 2000ms window monitors for secondary motion confirmation | ✅ Active |
| **Oldest Event Expiry** | If events are paired but outside 1500ms, oldest event is expired | ✅ Active |

### 2.4 Impact Classification — Axis-Based ✅

| Impact Type | Dominant Axis | Threshold (g) | Speed-Adjusted Range |
|-------------|---------------|---------------|---------------------|
| **Vertical Impact** | Z-axis | 1.2g | 0.84g – 1.56g |
| **Lateral Impact** | Y-axis | 1.0g | 0.70g – 1.30g |
| **Forward/Rear Impact** | X-axis | 0.8g | 0.56g – 1.04g |
| **Rollover** | Gravity shift | 7.0 m/s² | Fixed (no speed adjustment) |
| **General Impact** | Jerk-based | 2.0 g/s | Fixed |
| **Buffer Spike** | Pattern-based | (max-mean) > 1.2g | Fixed |

---

## 3. Threshold Calibration Strategy

### 3.1 Current Approach: Generic Hardcoded Thresholds ✅

All thresholds are tuned for average mid-range Android devices:
- G-force thresholds: Frontal 1.2g, Side 1.0g, Rollover 0.8g
- Sound threshold: -50.0 dBFS
- Gyro peak: 100 °/s
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

### 3.4 Future: Device-Profile Calibration (Phase 2)

Different devices have different sensor sensitivities (±20%). A future update may include:
- Auto-detect device model via `Build.MODEL`
- Apply device-specific sensitivity multipliers
- OR: User-facing calibration UI ("shake your phone to calibrate")

**Current recommendation:** Generic thresholds are sufficient for rule-based detection. Device profiles become important when ML model is integrated (training data must be normalized per device).

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

## 5. Acoustic Analysis — Current vs Ideal

### Current Implementation ✅
- **Metric:** dBFS (decibel full-scale) — overall loudness
- **Method:** RMS amplitude of PCM samples → dBFS conversion
- **Threshold:** > -50.0 dBFS triggers sound event
- **Calibration:** 1-second baseline at start
- **Polling:** 200ms intervals

### Ideal (Phase 2 — ML Refinement) 📋
- **Frequency Analysis:** FFT to extract spectral bands (crash noise has distinct 2–6 kHz signature)
- **Band Classification:**
  - 0–500 Hz: Engine rumble, road noise (ignore)
  - 500 Hz–2 kHz: Impact/collision (medium confidence)
  - 2–6 kHz: Glass breaking, metal impact, airbag (high confidence)
  - 6–10 kHz: Tire screech (contextual)
- **Benefits:** Distinguishes crash noise from music, conversation, wind
- **When:** Planned for ML pipeline (FeatureExtractor frequency-domain features)

> **Current verdict:** dBFS-only detection is adequate for rule-based fusion because it always requires a second correlated sensor event (impact, gyro, etc.). Frequency analysis will improve precision when ML model is integrated.

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
| Acoustic frequency analysis | 📋 0% | Phase 2 — dBFS sufficient for rule-based |
| Device-specific calibration | 📋 0% | Phase 2 — generic thresholds sufficient now |
| ML model inference | 📋 0% | Phase 2 — scaffolding ready |
| Background detection | 📋 0% | Requires foreground service |

### Overall Detection Level: **95% Complete (Rule-Based)**

> The remaining 5% covers acoustic frequency analysis and device-specific calibration profiles, which are **optimizations for ML phase** — not blockers for the current rule-based system.

---

## 8. What's Next for Detection (Roadmap)

| Phase | Milestone | Timeline | Impact |
|-------|-----------|----------|--------|
| **Phase 1** ✅ | Rule-based fusion (current) | Complete | Functional accident detection |
| **Phase 2** 📋 | Acoustic frequency bands (FFT) | Q3 2026 | Better crash vs noise distinction |
| **Phase 2** 📋 | Device-profile calibration | Q3 2026 | Tighter thresholds per device |
| **Phase 3** 📋 | TFLite model integration | Q4 2026 | ML-based classification |
| **Phase 3** 📋 | Background service | Q4 2026 | 24/7 detection |
| **Phase 4** 📋 | Emergency notification | Q1 2027 | Auto SOS + contacts |
| **Phase 4** 📋 | Cloud model updates | Q2 2027 | OTA model refinement |

---

**Last Updated:** March 2026  
**Author:** Anbu (@Anbu-2006)

