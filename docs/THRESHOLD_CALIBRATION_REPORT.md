# 📊 Healer App — Threshold Calibration Report

> ⚠️ **SUPERSEDED (March 2026):** Thresholds were recalibrated to crash-realistic levels.
> Current: Frontal 4.0g, Lateral 3.0g, Rollover 2.0g, Jerk 8.0 g/s, Gyro 300°/s.
> See `THRESHOLD_VALUES_REFERENCE.md` for authoritative current values.

**Last Updated:** March 2026  
**Status:** SUPERSEDED — see updated threshold reference docs

---

## 1. Accelerometer Thresholds (ShakeDetector)

### Base Thresholds (before speed/device adjustment)

| Impact Type | Threshold | Rationale |
|-------------|-----------|-----------|
| **Frontal (Z-axis)** | 1.2g | Car crash at 30 km/h produces ~0.8–1.0g on phone; 1.2g captures real crashes while filtering phone drops (<0.8g). Derived from NHTSA crash test telemetry scaled to consumer phone accelerometers. |
| **Lateral (Y-axis)** | 1.0g | Side impacts transfer less force through phone mount/pocket than head-on. Lower threshold captures T-bone collisions. |
| **Forward/Rear (X-axis)** | 0.8g | Rear-end collisions at low speed (fender-benders) produce lower peak G. 0.8g captures these while combined with pair requirement prevents false positives. |
| **Jerk** | 2.0 g/s | Crash onset is extremely rapid (0→peak in <100ms). Walking/bumps have gradual onset (<1.0 g/s). 2.0 g/s threshold captures crash-specific deceleration rate. |
| **Buffer Spike** | (max-mean) > 1.2g | A 20-sample buffer detects sustained impact events vs. single-sample noise spikes. If peak is 1.2g above mean, it indicates genuine shock event. |

### Speed-Gating Adjustments

| Speed Range | Multiplier | Adjusted Frontal | Adjusted Side | Adjusted Rollover | Rationale |
|-------------|-----------|-----------------|--------------|-------------------|-----------|
| < 3 km/h | ×1.3 | 1.56g | 1.30g | 1.04g | Stationary/walking — high bar to prevent drops triggering |
| 3–20 km/h | ×1.0 | 1.20g | 1.00g | 0.80g | Urban — baseline sensitivity |
| 20–50 km/h | ×0.85 | 1.02g | 0.85g | 0.68g | Suburban — slightly more sensitive |
| > 50 km/h | ×0.7 | 0.84g | 0.70g | 0.56g | Highway — most sensitive (high-speed = severe) |

### Device Profile Adjustments

| Device Tier | Multiplier | Effect on Frontal | Rationale |
|-------------|-----------|------------------|-----------|
| Flagship (Samsung S24, Pixel 8) | ×0.90 | 1.08g | High-quality MEMS sensors report higher values → raise threshold slightly |
| Mid-range (OnePlus Nord, Moto G) | ×1.0 | 1.20g | Average sensors → baseline |
| Budget (Redmi Note, Samsung A10) | ×1.10–1.15 | 1.32–1.38g | Lower-quality sensors have more noise → slightly raise threshold to reduce false triggers |

---

## 2. Sound Threshold (SoundProvider)

| Parameter | Value | Rationale |
|-----------|-------|-----------|
| **Threshold** | -50.0 dBFS | Car crash produces -40 to -20 dBFS at phone microphone. Normal conversation is -60 to -50 dBFS. -50.0 captures loud impacts while filtering normal speech. |
| **Baseline Cap** | -3.0 dBFS | Prevents calibration in extremely noisy environments from setting an unreachably high baseline. |
| **Calibration Window** | 1000ms | 1 second is enough to establish ambient noise floor but short enough for quick app startup. |
| **Polling Interval** | 200ms | 5 Hz sampling balances responsiveness (crash detection within 200ms) with battery usage. |
| **Sample Rate** | 44100 Hz | CD-quality; required for accurate FFT up to 22 kHz (Nyquist). Standard for Android AudioRecord. |

### Audio Frequency Bands (AudioSignalProcessor)

| Band | Frequency Range | Weight | Typical Source |
|------|----------------|--------|----------------|
| Band 0 | 0–500 Hz | 0.2× | Engine rumble, road noise, wind — NOT crash-indicative |
| Band 1 | 500–2000 Hz | 0.5× | Heavy impact, body panel deformation — moderate indicator |
| Band 2 | 2000–6000 Hz | 1.0× | Glass breaking, metal crumpling, airbag — **strongest crash indicator** |
| Band 3 | 6000–10000 Hz | 0.3× | Tire screech, high-pitched debris — contextual support |

**Crash Signature Score:** Weighted sum of normalized band energies (0.0–1.0). Score > 0.45 = crash-like audio.

---

## 3. Gyroscope Threshold (GyroProvider)

| Parameter | Value | Rationale |
|-----------|-------|-----------|
| **Peak Threshold** | 100 °/s | Sudden vehicle spin or flip exceeds 100°/s. Normal phone handling (placing on table, picking up) rarely exceeds 60°/s. 100°/s provides clear separation. |
| **Device-Adjusted** | 100 × deviceProfile.gyroMultiplier | Budget gyroscopes have ±10% sensitivity variance. Profile multiplier compensates. |

---

## 4. Barometer Threshold (BarometerProvider)

| Parameter | Value | Rationale |
|-----------|-------|-----------|
| **Spike Threshold** | ≥ 0.5 hPa | Car cabin pressure changes by 0.5–2.0 hPa on window break or structural deformation. Normal altitude changes produce <0.1 hPa/s. 0.5 hPa in 100ms is a reliable impact signature. |
| **Spike Window** | 100ms | Pressure change from impact occurs within milliseconds. 100ms window captures this while filtering slow weather/altitude changes. |
| **Debounce** | 2000ms | Prevents repeated triggers from pressure oscillation after initial spike. |

---

## 5. Gravity/Rollover Threshold (GravityProvider)

| Parameter | Value | Rationale |
|-----------|-------|-----------|
| **Rollover Shift** | > 7.0 m/s² | Earth gravity is 9.81 m/s². A 7.0 m/s² shift on any axis means the device has rotated ~45°+ rapidly — consistent with vehicle rollover. Normal phone repositioning produces <3.0 m/s² shifts. |
| **Baseline Samples** | 20 | 20 samples (~400ms) provides stable baseline orientation at session start. |
| **Debounce** | 3000ms | Vehicle rollovers involve multiple rotations; 3s debounce captures the event once. |

---

## 6. Light Threshold (LightProvider)

| Parameter | Value | Rationale |
|-----------|-------|-----------|
| **Sudden Darkness** | < 30% of baseline | If ambient light drops to <30% of calibrated baseline AND >50% single-reading drop, it indicates sudden obstruction (airbag, phone flung under seat). Normal indoor dimming rarely drops below 40%. |
| **Baseline Samples** | 10 | 10 readings (~1s) establishes ambient light level. |
| **Confidence Boost** | +10 (very dark) / +5 (moderate) | Supplementary signal, not standalone. Adds certainty when paired with impact/sound. |

---

## 7. Proximity Threshold (ProximityProvider)

| Parameter | Value | Rationale |
|-----------|-------|-----------|
| **NEAR Detection** | distance < maxRange | Binary sensor on most Android devices. NEAR = phone in pocket/against body/covered by debris. |
| **Confidence Boost** | +15 (NEAR) / +0 (FAR) | Phone in pocket during impact is strong accident indicator (user wasn't actively using phone). Exposed phone may indicate dropped phone (false positive). |

---

## 8. Activity Detection Thresholds (ActivityDetector)

| Parameter | Value | Rationale |
|-----------|-------|-----------|
| **Walking** | > 30 steps/min | Average walking pace is 100–120 steps/min. 30 steps/min threshold captures even slow walking while filtering momentary step-counter triggers. |
| **Vehicle Speed** | > 5 m/s (18 km/h) | Below 18 km/h could be brisk cycling or running. Above 18 km/h strongly suggests motorized vehicle. |
| **Vehicle Fallback** | > 1 m/s without steps | Moving without stepping = in vehicle (even at slow speed like parking lot). |

### Activity Multipliers

| State | Multiplier | Effect on Score | Rationale |
|-------|-----------|----------------|-----------|
| IN_VEHICLE | ×1.5 | 50% boost | Most accidents are vehicular; high confidence when confirmed in vehicle. |
| STILL | ×1.0 | No change | Baseline — could be parked car or pedestrian. |
| WALKING | ×0.3 | 70% reduction | Walking + impact = likely phone drop. Strong suppression prevents false alerts. |
| UNKNOWN | ×1.0 | No change | Insufficient data — use baseline. |

---

## 9. Fusion Window

| Parameter | Value | Rationale |
|-----------|-------|-----------|
| **Correlation Window** | 1500ms | Real accidents trigger multiple sensors within 1-2 seconds (impact → sound → gyro → pressure). 1500ms captures this chain while filtering unrelated events (e.g., impact from bump + unrelated music 5 seconds later). |
| **Pair Requirement** | 2+ events | Single-sensor events have high false-positive rate. Requiring 2+ correlated events within 1500ms dramatically reduces false positives while maintaining sensitivity. |

---

## 10. Validation Test Cases

| # | Scenario | Expected G | Expected dB | Expected Result | Notes |
|---|----------|-----------|------------|----------------|-------|
| 1 | Car crash at 30 km/h | 0.8–1.5g | -40 to -25 dBFS | ✅ DETECT | Impact + Sound pair within 1500ms |
| 2 | Car crash at 60 km/h | 1.5–3.0g | -30 to -15 dBFS | ✅ DETECT | High G + loud sound + likely gyro spin |
| 3 | Vehicle rollover | 0.5–1.0g sustained | Variable | ✅ DETECT | Gravity shift + gyro spin pair |
| 4 | Phone drop from hand | 0.4–0.7g | -55 to -45 dBFS | ❌ IGNORE | Below threshold, walking suppression |
| 5 | Phone drop on table | 0.6–1.0g | -50 to -40 dBFS | ❌ IGNORE | No second sensor pair (no light/proximity change) |
| 6 | Rough road bump | 0.3–0.5g | < -60 dBFS | ❌ IGNORE | Below threshold, no sound correlation |
| 7 | Speed bump at 30 km/h | 0.4–0.8g | < -60 dBFS | ❌ IGNORE | Below adjusted threshold, no pair |
| 8 | Loud music in car | < 0.1g | -30 to -10 dBFS | ❌ IGNORE | No impact pair, single-channel |
| 9 | Walking + stumble | 0.5–0.9g | < -55 dBFS | ❌ IGNORE | Walking ×0.3 multiplier + no pair |
| 10 | Cyclist fall at 20 km/h | 0.8–1.5g | -50 to -35 dBFS | ✅ DETECT | Impact + sound, speed-gated thresholds |

---

**Last Updated:** March 2026  
**Author:** Anbu (@Anbu-2006)

