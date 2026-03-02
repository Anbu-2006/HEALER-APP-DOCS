# 🩺 Healer App

![Project Status](https://img.shields.io/badge/Status-In%20Development%20(85%25)-yellow)
![License](https://img.shields.io/badge/License-Proprietary-red)
![Platform](https://img.shields.io/badge/Platform-Android-green)
![Language](https://img.shields.io/badge/Language-Kotlin-purple)
![Min SDK](https://img.shields.io/badge/Min%20SDK-24%20(Android%207.0)-blue)
![ML](https://img.shields.io/badge/ML-TFLite%20(Planned)-orange)

An intelligent, multi-sensor accident detection application for Android that fuses data from **10+ hardware sensors** to detect potential vehicle accidents, falls, and emergencies — and provide timely assistance.

---

## 📋 Project Status

**Current Progress: 85% Complete**

This project has completed a major architectural refactoring to separate sensor acquisition from detection logic, establishing a clean ML-ready foundation. Core detection features are fully functional. Machine Learning integration and background service support are the next major milestones.

| Milestone | Status |
|-----------|--------|
| Core sensor providers (data acquisition) | ✅ Complete |
| BaseSensor interface abstraction | ✅ Complete |
| Modular detection layer (pure logic) | ✅ Complete |
| Sensor fusion engine (DetectionCoordinator) | ✅ Complete |
| Rule-based accident detection | ✅ Complete |
| UI & live sensor dashboard | ✅ Complete |
| ML feature extraction pipeline | 🔧 In Progress |
| Circular sensor buffer (2s sliding window) | ✅ Complete |
| Training data logger (CSV export) | ✅ Complete |
| TFLite on-device inference | 📋 Planned |
| Background detection service | 📋 Planned |
| Emergency contact notification | 📋 Planned |
| Cloud backup & model updates | 📋 Planned |
| Public source code release | 📋 June 2027 |

---

## ✨ Key Features

| Feature | Sensor Used | Description | Status |
|---------|-------------|-------------|--------|
| 🔊 **Sound Detection** | Microphone (44.1 kHz PCM) | Monitors ambient sound in dBFS; detects loud crashes/impacts; auto-calibrates baseline | ✅ Implemented |
| 📳 **Shake Detection** | Accelerometer / Linear Acceleration | Detects sudden G-force spikes; classifies Frontal, Lateral, Vertical, Rollover impacts; jerk detection (Δg/s) | ✅ Implemented |
| 🌍 **Gravity Sensor** | TYPE_GRAVITY | Tracks device orientation relative to Earth; detects rollover (>7 m/s² shift); auto-calibrates baseline | ✅ Implemented |
| 📍 **Location Provider** | GPS / Network | Tracks speed (m/s, km/h) for speed-gated thresholds; provides last-known location for emergency use | ✅ Implemented |
| 🔄 **Gyroscope Detection** | TYPE_GYROSCOPE | Analyzes rotational velocity (°/s); peak detection for sudden spins; feeds fusion engine | ✅ Implemented |
| 💡 **Light Sensor** | TYPE_LIGHT | Monitors ambient lux; detects sudden darkness (airbag, phone thrown under seat); auto-calibrates | ✅ Implemented |
| 📱 **Proximity Sensor** | TYPE_PROXIMITY | Detects if phone is in pocket/covered vs exposed; boosts confidence if covered during impact | ✅ Implemented |
| 🧭 **Orientation Helper** | TYPE_ROTATION_VECTOR | Provides world-frame rotation matrix; transforms device-frame acceleration to world-frame | ✅ Implemented |
| 🚶 **Activity Detection** | TYPE_STEP_COUNTER + GPS | Classifies STILL / WALKING / IN_VEHICLE; suppresses false positives from walking | ✅ Implemented |
| 🌡️ **Barometer** | TYPE_PRESSURE | Pressure spike detection (Δ ≥ 0.5 hPa in <100ms); high-confidence impact trigger | ✅ Implemented |
| 🎯 **Sensor Fusion** | All sensors combined | 6-channel event correlation within 1500ms; weighted confidence scoring (0–100%) | ✅ Implemented |
| 📊 **Training Logger** | CSV export | Logs timestamped sensor data for ML training; session-based CSV files | ✅ Implemented |
| 🤖 **ML Classifier** | TensorFlow Lite | On-device accident classification from sensor features | 📋 Planned |

---

## 🔒 Source Code Availability

> **⚠️ IMPORTANT NOTICE**
>
> **This project is currently Closed Source for security reasons.**
>
> The source code will be released in **JUNE 2027** (previously December 2026, extended for ML refinement and background service implementation).
>
> This repository contains **documentation and architecture designs only**.
>
> We appreciate your understanding as we ensure the security and stability of the application before making the codebase publicly available.

---

## 🏗️ Current Project Structure (After Refactoring)

```
app/
└── src/main/
    ├── AndroidManifest.xml
    ├── java/com/example/healer/
    │   ├── MainActivity.kt              # Entry point — Start button → SecondActivity
    │   ├── SecondActivity.kt            # Main detector screen — permissions, UI callbacks only
    │   │
    │   ├── sensors/                     # ── Sensor Data Acquisition Layer ──
    │   │   ├── BaseSensor.kt            # Interface: hasSensor(), start(), stop(), isRunning()
    │   │   ├── AccelProvider.kt         # Accelerometer → world-frame acceleration (m/s²)
    │   │   ├── GyroProvider.kt          # Gyroscope → angular velocity (rad/s)
    │   │   ├── GravityProvider.kt       # Gravity sensor → Earth frame orientation
    │   │   ├── LightProvider.kt         # Light sensor → ambient lux + darkness detection
    │   │   ├── ProximityProvider.kt     # Proximity sensor → pocket/near detection
    │   │   ├── SoundProvider.kt         # Microphone → dBFS levels
    │   │   ├── BarometerProvider.kt     # Barometer → pressure spike detection
    │   │   ├── LocationProvider.kt      # GPS → speed & coordinates
    │   │   ├── OrientationHelper.kt     # Rotation vector → 3×3 matrix (device-to-world)
    │   │   └── ActivityDetector.kt      # Step counter + GPS → STILL/WALKING/IN_VEHICLE
    │   │
    │   ├── detection/                   # ── Detection & Fusion Logic Layer ──
    │   │   ├── ShakeDetector.kt         # Impact classification (pure logic, no sensor listener)
    │   │   ├── DetectionCoordinator.kt  # Centralized fusion engine, time-window correlation
    │   │   ├── DetectionCapabilities.kt # Runtime sensor capability scanner
    │   │   └── HighPassFilter.kt        # Math utility: gravity removal for raw accelerometer
    │   │
    │   ├── data/                        # ── Data Persistence & Buffering ──
    │   │   ├── CircularSensorBuffer.kt  # Thread-safe 2-second sliding window ring buffer
    │   │   ├── SensorSnapshot.kt        # Single timestamped multi-sensor sample
    │   │   └── TrainingLogger.kt        # CSV logger for ML training data (world-frame normalized)
    │   │
    │   └── ml/                          # ── ML Pipeline (Scaffolding) ──
    │       └── FeatureExtractor.kt      # Flattens 2s buffer → ML-ready feature vector
    │
    └── res/
        ├── layout/
        │   ├── page_1.xml               # Home screen (Start button)
        │   └── page_2.xml               # Detector dashboard (sensor readouts + alerts)
        ├── anim/                        # Fade/slide transitions
        ├── drawable/                    # Launcher icons
        ├── values/                      # Colors, dimens, strings, themes
        └── xml/                         # Backup & data extraction rules
```

**Key Separation:**
- `sensors/` = **Pure data acquisition** — all implement `BaseSensor`, no detection logic, callbacks deliver raw samples
- `detection/` = **Pure logic** — no sensor listeners, receives processed data from `sensors/`, implements fusion & classification
- `data/` = **Persistence** — circular buffer, snapshots, training logger (shared between layers)
- `ml/` = **ML scaffolding** — feature extraction, model placeholder for TFLite integration

---

## 🏗️ Project Structure (Refactored ML-Ready Architecture)

```
app/
└── src/main/
    ├── AndroidManifest.xml
    ├── java/com/example/healer/
    │   ├── MainActivity.kt                      # Entry point, navigation
    │   ├── SecondActivity.kt                    # Detector dashboard, UI callbacks only
    │   │
    │   ├── sensors/                             # ── Sensor Data Acquisition Layer ──
    │   │   ├── BaseSensor.kt                    # Interface contract for all sensors
    │   │   ├── AccelProvider.kt                 # Accel → world-frame via OrientationHelper
    │   │   ├── GyroProvider.kt                  # Gyro → angular velocity (rad/s)
    │   │   ├── GravityProvider.kt               # Gravity → orientation + rollover detection
    │   │   ├── SoundProvider.kt                 # Sound → dBFS levels + auto-calibration
    │   │   ├── LightProvider.kt                 # Light → lux + sudden darkness
    │   │   ├── ProximityProvider.kt             # Proximity → near/far pocket state
    │   │   ├── BarometerProvider.kt             # Barometer → pressure spike (optional)
    │   │   ├── LocationProvider.kt              # Location → GPS speed + coordinates
    │   │   ├── OrientationHelper.kt             # RotationVector → 3×3 rotation matrix
    │   │   └── ActivityDetector.kt              # StepCounter + speed → activity state
    │   │
    │   ├── detection/                           # ── Detection & Fusion Logic Layer ──
    │   │   ├── ShakeDetector.kt                 # Impact classification (pure logic)
    │   │   ├── DetectionCoordinator.kt          # Centralized fusion + time-window correlation
    │   │   ├── DetectionCapabilities.kt         # Runtime hardware capability check
    │   │   ├── DeviceProfile.kt                 # Per-device sensor sensitivity data
    │   │   ├── DeviceProfileRegistry.kt         # Auto-detect device → calibration profile (30+)
    │   │   └── HighPassFilter.kt                # Math utility: gravity removal
    │   │
    │   ├── data/                                # ── Data Persistence Layer ──
    │   │   ├── CircularSensorBuffer.kt          # Thread-safe 2s sliding window buffer
    │   │   ├── SensorSnapshot.kt                # Timestamped multi-sensor sample
    │   │   └── TrainingLogger.kt                # CSV logger (world-frame normalized data)
    │   │
    │   ├── ml/                                  # ── ML Pipeline (In Progress) ──
    │   │   ├── FeatureExtractor.kt              # Buffer → feature vector (48 features, TFLite-ready)
    │   │   ├── AudioSignalProcessor.kt          # FFT frequency-band crash signature analysis
    │   │   ├── SignalProcessor.kt               # Noise filtering, normalization (planned)
    │   │   └── [AccidentClassifier.kt]          # TFLite model runner (planned)
    │   │
    │   └── service/                             # ── Background Service (Planned) ──
    │       └── [DetectionService.kt]            # Foreground service for 24/7 detection
    │
    └── res/
        ├── layout/
        │   ├── page_1.xml                       # Home screen
        │   └── page_2.xml                       # Detector dashboard
        ├── anim/
        │   ├── fade_in.xml
        │   ├── fade_out.xml
        │   ├── slide_in_bottom.xml
        │   └── slide_out_bottom.xml
        ├── drawable/
        │   ├── ic_launcher_background.xml
        │   └── ic_launcher_foreground.xml
        ├── mipmap-*/
        │   ├── ic_launcher.webp
        │   └── ic_launcher_round.webp
        ├── values/
        │   ├── colors.xml
        │   ├── dimens.xml
        │   ├── strings.xml
        │   ├── styles.xml
        │   └── themes.xml
        └── xml/
            ├── backup_rules.xml
            └── data_extraction_rules.xml
```

**Architecture Highlights:**
- ✅ **Sensor/Logic Separation** — `sensors/` acquires raw data, `detection/` processes it
- ✅ **Interface Abstraction** — `BaseSensor` unifies sensor lifecycle management
- ✅ **Centralized Fusion** — `DetectionCoordinator` orchestrates all providers & detectors
- ✅ **ML-Ready Buffer** — `CircularSensorBuffer` maintains 2s sliding window for feature extraction
- ✅ **World-Frame Normalization** — `OrientationHelper` provides device-to-Earth rotation matrix
- 🔧 **Feature Pipeline** — `FeatureExtractor` scaffolding ready for TFLite integration
- 📋 **Planned** — Background service, signal processor, ML classifier

---

## 📦 Module-by-Module Breakdown

### 1. **UI Layer** — Pure Presentation

#### `MainActivity.kt`
| Aspect | Detail |
|--------|--------|
| **Purpose** | Entry screen with single CTA button |
| **Layout** | `page_1.xml` — centered "Start Accident Detection" button |
| **Action** | Navigates to `SecondActivity` with `FLAG_ACTIVITY_SINGLE_TOP`, then `finish()` |
| **Lifecycle** | Minimal — no sensor work, no permissions |

#### `SecondActivity.kt` (UI-Only Refactored)
| Aspect | Detail |
|--------|--------|
| **Purpose** | Main accident detection dashboard |
| **Layout** | `page_2.xml` — ScrollView with 18+ live sensor TextViews |
| **Implements** | `DetectionListener` interface (callbacks for UI updates only) |
| **Permissions** | Required: `RECORD_AUDIO`. Optional: `ACCESS_FINE_LOCATION`, `ACCESS_COARSE_LOCATION` |
| **Lifecycle** | `onCreate` → bind views. `onResume` → resume coordinator. `onPause` → pause coordinator. `onDestroy` → stop & cleanup |
| **Responsibilities** | ✅ Permission handling, ✅ View binding, ✅ UI updates, ❌ NO detection logic |

**Key Methods:**
- `initializeViews()` — Binds 18+ TextViews and button click listeners
- `resetUI(initial)` — Resets all displays to defaults
- `requestStartPermissions()` — Permission dialog flow
- `startDetection()` — Creates & starts coordinator
- `stopDetection()` — Stops coordinator & cleans state
- `onAccelUpdate(x,y,z,gForce,type)` — Updates accel/impact display
- `onGyroUpdate(x,y,z,mag)` — Updates rotation display
- `onSoundUpdate(db)` — Updates sound level & color
- `onGravityUpdate(x,y,z,orient)` — Updates gravity & orientation
- `onProximityUpdate(isNear, distance)` — Updates proximity display
- `onLightUpdate(lux, condition, sudden)` — Updates light display
- `onActivityUpdate(activity, steps)` — Updates activity state
- `onSpeedUpdate(mps, kmh)` — Updates speed display
- `onConfidenceUpdate(breakdown)` — Updates per-sensor scores
- `onStatusUpdate(text, severity)` — Updates fusion status & colors
- `onProbabilityUpdate(prob)` — Updates accident probability %
- `onAccidentConfirmed(...)` — Shows alert dialog + vibrates
- `showAccidentAlert()` — Modal dialog with sensor summary
- `vibrateDevice()` — Waveform feedback

---

### 2. **sensors/** — Data Acquisition Layer (Pure Input)

All sensor wrappers implement `BaseSensor` interface. They **acquire raw data only**, with no detection logic.

#### `BaseSensor.kt` (Interface)
```kotlin
interface BaseSensor {
    val tag: String
    fun hasSensor(): Boolean
    fun start()
    fun stop()
    fun isRunning(): Boolean
}
```

#### `AccelProvider.kt` — Accelerometer (World-Frame)
| Aspect | Detail |
|--------|--------|
| **Sensor** | `TYPE_LINEAR_ACCELERATION` (preferred) or `TYPE_ACCELEROMETER` + HighPassFilter |
| **Rate** | `SENSOR_DELAY_GAME` (~200Hz), throttled to 20ms min |
| **Processing** | Raw → filter if needed → `OrientationHelper.rotateToWorld()` → world-frame m/s² |
| **Callback** | `onAccelData(wx, wy, wz, timestampMs)` |
| **To** | `DetectionCoordinator` processes via `ShakeDetector.processAccelData()` |

#### `GyroProvider.kt` — Gyroscope
| Aspect | Detail |
|--------|--------|
| **Sensor** | `TYPE_GYROSCOPE` at `SENSOR_DELAY_GAME` |
| **Output** | (x, y, z) in rad/s, magnitude |
| **Callback** | `onGyroData(x, y, z, mag, timestampMs)` |
| **Logic** | None — raw values only |

#### `GravityProvider.kt` — Gravity Orientation
| Aspect | Detail |
|--------|--------|
| **Sensor** | `TYPE_GRAVITY` at `SENSOR_DELAY_GAME` |
| **Calibration** | 20-sample baseline for orientation baseline |
| **Rollover Detection** | Axis shift >7 m/s² triggers `onRolloverDetected()` callback |
| **Orientation String** | FLAT_FACE_UP/DOWN, PORTRAIT_UP/DOWN, LANDSCAPE_LEFT/RIGHT |
| **Callbacks** | `onGravityData(x,y,z,orient,ts)`, `onRolloverDetected(from,to)` |
| **Confidence Methods** | `getConfidenceBoost()` → 0-20 points based on orientation change |

#### `SoundProvider.kt` — Microphone
| Aspect | Detail |
|--------|--------|
| **Source** | `MediaRecorder.AudioSource.MIC` via `AudioRecord` |
| **Config** | 44100 Hz, MONO, PCM_16BIT, buffer-based polling (200ms) |
| **Metric** | dBFS = 20 × log10(RMS / 32768) |
| **Calibration** | 1-second auto-baseline (capped at -3.0 dBFS) |
| **Callback** | `onSoundData(db, timestampMs)` |

#### `LightProvider.kt` — Ambient Light
| Aspect | Detail |
|--------|--------|
| **Sensor** | `TYPE_LIGHT` at `SENSOR_DELAY_NORMAL` |
| **Calibration** | 10-sample baseline lux |
| **Sudden Darkness** | `<30% baseline AND drops >50%` → `onSuddenDarkness()` |
| **Conditions** | Dark/Dim/Indoor/Bright Indoor/Outdoor/Sunlight (method: `getLightCondition()`) |
| **Confidence Methods** | `getConfidenceBoost()` → 0-10 points |
| **Callbacks** | `onLightData(lux, sudden, ts)`, `onSuddenDarkness()` |

#### `ProximityProvider.kt` — Pocket Detection
| Aspect | Detail |
|--------|--------|
| **Sensor** | `TYPE_PROXIMITY` at `SENSOR_DELAY_NORMAL` |
| **Logic** | Binary: `distance < maxRange` → NEAR (pocket), else FAR |
| **Callback** | `onProximityData(isNear, distance, ts)` |
| **Confidence** | `getConfidenceBoost()` → 15 if NEAR (in pocket = more likely accident), 0 if FAR |

#### `BarometerProvider.kt` — Atmospheric Pressure
| Aspect | Detail |
|--------|--------|
| **Sensor** | `TYPE_PRESSURE` at `SENSOR_DELAY_FASTEST` |
| **Spike Detection** | Δ ≥ 0.5 hPa within 100ms → `onPressureSpike()` (cabin pressure = impact) |
| **Debounce** | 2000ms between spike alerts |
| **Callbacks** | `onPressureData(hPa, ts)`, `onPressureSpike(delta)` |

#### `LocationProvider.kt` — GPS Position & Speed
| Aspect | Detail |
|--------|--------|
| **Source** | `LocationManager` — GPS_PROVIDER (preferred) or NETWORK_PROVIDER fallback |
| **Updates** | 1000ms / 1m minimum distance |
| **Methods** | `lastSpeedMps()`, `lastLocation` property |
| **Permissions** | `ACCESS_FINE_LOCATION` or `ACCESS_COARSE_LOCATION` |

#### `OrientationHelper.kt` — World-Frame Rotation
| Aspect | Detail |
|--------|--------|
| **Sensor** | `TYPE_ROTATION_VECTOR` at `SENSOR_DELAY_GAME` |
| **Maintains** | 3×3 rotation matrix (device-to-world transform) |
| **Method** | `rotateToWorld(ax, ay, az)` → FloatArray[3] (world coordinates) |
| **Used By** | `AccelProvider` to normalize accelerometer to world frame |

#### `ActivityDetector.kt` — Motion State (STILL/WALKING/IN_VEHICLE)
| Aspect | Detail |
|--------|--------|
| **Sensors** | `TYPE_STEP_COUNTER` + external GPS speed |
| **States** | UNKNOWN, STILL, WALKING (>30 steps/min), IN_VEHICLE (>5 m/s) |
| **Multiplier** | VEHICLE=1.5×, STILL=1.0×, WALKING=0.3× (suppress false positives), UNKNOWN=1.0× |
| **Methods** | `updateActivity()`, `getTotalSteps()`, `getConfidenceMultiplier()` |
| **Callback** | `onActivityChange(activity)` |

---

### 3. **detection/** — Logic & Fusion Layer (Pure Processing)

No sensor listeners here — all data comes from `sensors/` providers.

#### `ShakeDetector.kt` — Impact Classification (Pure Logic)
| Aspect | Detail |
|--------|--------|
| **Input** | World-frame acceleration from `AccelProvider` via `processAccelData(wx, wy, wz, ts)` |
| **Processing** | G-force conversion, jerk calculation, speed-gated thresholds, impact type classification |
| **Thresholds (g)** | Frontal: 1.2, Side: 1.0, Rollover: 0.8 (speed-adjusted: <3km/h ×1.3, >50km/h ×0.7) |
| **Jerk** | Δg/s ≥ 2.0 triggers detection |
| **Buffer** | 20-sample g-force rolling buffer; spike = (max - mean) > 1.2g |
| **Post-Impact** | 2000ms confirmation window for secondary motion |
| **Output** | `onCrashDetected(type, gForce, location)`, `onImpactValues(x,y,z,gForce,type)`, `onError(msg)` |
| **No Sensor Listener** | ✅ Pure logic layer |

#### `DetectionCoordinator.kt` — Centralized Fusion Engine
| Aspect | Detail |
|--------|--------|
| **Purpose** | Orchestrates all 10+ sensor providers, coordinates timing, applies fusion logic |
| **Owns** | All provider instances, `ShakeDetector`, `CircularSensorBuffer`, `TrainingLogger` |
| **Thread** | Dedicated `HandlerThread` for sensor processing |
| **Event Correlation** | Tracks 6 timestamps (shake, sound, gyro, light darkness, rollover, barometer spike) |
| **Fusion Window** | 1500ms time window — any 2 events within window = potential accident |
| **Activity Check** | If WALKING detected → suppress alert (false positive) |
| **Probability Calc** | Weighted: Impact(30) + GForce(25) + Sound(15) + Proximity(0-15) + Light(0-10) + Gravity(0-20) × Activity multiplier |
| **Callbacks** | Implements `DetectionListener` — delivers all data to SecondActivity UI |
| **Lifecycle** | `start()` → init all providers, `stop()` → cleanup, `pause()`/`resume()` for onPause/onResume |

#### `DetectionCapabilities.kt` — Runtime Hardware Check
| Aspect | Detail |
|--------|--------|
| **Scans** | 11 sensor capabilities: Accelerometer, Gyroscope, RotationVector, LinearAcceleration, Gravity, Proximity, Light, StepCounter, Barometer, Microphone, GPS |
| **Score** | `capabilityScore()` = (available / 11) × 100% |
| **UI** | AlertDialog before detection starts |
| **Graceful** | App skips missing sensors, still works with available hardware |

#### `HighPassFilter.kt` — Gravity Removal Math Utility
| Aspect | Detail |
|--------|--------|
| **Algorithm** | Exponential smoothing: `gravity_est = α × prev + (1-α) × raw`, output = `raw - gravity_est` |
| **Alpha** | 0.9 (slow adaptation) |
| **Used By** | `AccelProvider` when device lacks `TYPE_LINEAR_ACCELERATION` |

---

### 4. **data/** — Persistence & Buffering

#### `CircularSensorBuffer.kt` — 2-Second Sliding Window
| Aspect | Detail |
|--------|--------|
| **Capacity** | 100 samples (~2 seconds at 50 Hz) |
| **Thread-Safe** | ReadWriteLock (many readers, exclusive writer) |
| **Methods** | `put(snapshot)`, `getAll()`, `latest()`, `getWindow(ms)`, `size()`, `isFull()`, `clear()` |
| **Used By** | `DetectionCoordinator` (writes), `FeatureExtractor` (reads) |

#### `SensorSnapshot.kt` — Multi-Sensor Sample
| Aspect | Detail |
|--------|--------|
| **Fields** | timestamp, accel (world-frame X/Y/Z + gForce), gyro (X/Y/Z), sound (dBFS), gravity (X/Y/Z), proximity (bool), light (lux), speed (m/s), activity state, barometer (hPa), impact type |
| **Stored By** | `DetectionCoordinator` continuously into `CircularSensorBuffer` |

#### `TrainingLogger.kt` — ML Training Data (CSV)
| Aspect | Detail |
|--------|--------|
| **Format** | Append-only CSV, session-per-file |
| **Location** | `Android/data/com.example.healer/files/logs/session_<timestamp>.csv` |
| **Schema** | `timestamp, uptime_ms, acc_x, acc_y, acc_z, g_force, dbfs, gyro_x, gyro_y, gyro_z, speed_mps, activity_state, impact_type, event` |
| **Normalization** | Acceleration is world-frame (normalized by `OrientationHelper`) |
| **Events** | `sensor` (continuous), `sound` (threshold), `gyro` (rotation), `crash` (confirmed) |
| **Threading** | Single-thread executor, buffered writes (flush every 32 KB) |

---

### 5. **ml/** — ML Pipeline (In Progress)

#### `FeatureExtractor.kt` — Buffer → Feature Vector
| Aspect | Detail |
|--------|--------|
| **Input** | `CircularSensorBuffer` (list of `SensorSnapshot`) |
| **Window** | 2.0 seconds of data, minimum 10 samples |
| **Time-Domain Features** | Mean, std, min, max, RMS, zero-crossing-rate per axis (accel X/Y/Z, gyro X/Y/Z) |
| **Frequency-Domain** | (Planned) FFT energy bands, dominant frequency, spectral entropy |
| **Cross Features** | (Planned) Inter-axis correlation, magnitude deltas, jerk statistics |
| **Context Features** | (Planned) GPS speed, activity state, proximity, light level |
| **Output** | FloatArray (45 features planned) |
| **Status** | Scaffolding complete, feature list ready for TFLite integration |

---

### 6. **service/** — Background Service (Planned)

#### `DetectionService.kt` (Planned)
| Aspect | Detail |
|--------|--------|
| **Purpose** | Foreground Service to maintain detection when app is backgrounded |
| **Notification** | Persistent "Accident Detection Active" notification |
| **Lifecycle** | Started by SecondActivity, survives onPause/onStop, stopped explicitly |
| **Contains** | Own instances of all sensor providers + ML classifier |

---

## 🔌 Data Flow (Refactored Architecture)

```
┌──────────────────────────────────────────────────────────────────┐
│               HARDWARE SENSORS (OS level)                        │
│  Accelerometer  Gyroscope  Gravity  Light  Proximity  Mic  GPS   │
└───┬──────────┬──────────┬────────┬───────┬─────────┬────┬───────┘
    │          │          │        │       │         │    │
    ▼          ▼          ▼        ▼       ▼         ▼    ▼
 ┌──────────┐ ┌─────────┐ ┌──────┐ ┌───┐ ┌─────┐ ┌───┐ ┌────┐
 │Accel     │ │Gyro     │ │Grav  │ │Lgt│ │Prox │ │Snd│ │GPS │
 │Provider  │ │Provider │ │Prov  │ │Pr│ │Prov │ │Pr│ │Prov│
 └────┬─────┘ └────┬────┘ └──┬───┘ └─┬┘ └──┬──┘ └──┬┘ └─┬──┘
      │            │         │       │     │       │    │
      │ ┌──────────────────────────────────────────────┐ │
      │ │    OrientationHelper (RotationVector)        │ │
      │ │    Maintains 3×3 device-to-world matrix     │ │
      │ └──────────┬──────────────────────────────────┘ │
      │            │ (world-frame rotation)             │
      ▼            ▼              ▼       ▼       ▼     ▼
┌──────────────────────────────────────────────────────────────────┐
│           DetectionCoordinator (sensors/ → fusion)               │
│                                                                  │
│  All providers registered with callbacks                         │
│  └─ AccelProvider.onAccelData  ──→ ShakeDetector.processAccel    │
│  └─ GyroProvider.onGyroData    ──→ Track gyroDetectedTime       │
│  └─ GravityProvider.onGravity  ──→ Detect rollover              │
│  └─ SoundProvider.onSoundData  ──→ Track soundDetectedTime      │
│  └─ LightProvider.onLightData  ──→ Detect sudden darkness       │
│  └─ ProximityProvider.onProximity ──→ Get confidence boost      │
│  └─ BarometerProvider.onPressure  ──→ Detect pressure spike     │
│  └─ LocationProvider.lastSpeedMps() ──→ Speed gating            │
│  └─ ActivityDetector.currentActivity ──→ Activity multiplier    │
│  └─ OrientationHelper.rotateToWorld() ──→ World-frame norm      │
│                                                                  │
│  Fusion Logic:                                                   │
│  ├─ Track 6 event timestamps (shake, sound, gyro, light,       │
│  │  rollover, barometer)                                        │
│  ├─ hasPairWithinWindow(1500ms)?  ──→ Any 2 events near?       │
│  ├─ isLikelyWalking()?  ──→ Suppress false positive             │
│  ├─ calculateProbability() ──→ Weighted sum → 0-100%           │
│  └─ Output → SecondActivity via DetectionListener callbacks    │
│                                                                  │
│  Also:                                                           │
│  └─ CircularSensorBuffer.put(snapshot) ──→ ML feature input    │
│  └─ TrainingLogger.logSensor(...) ──→ CSV training data        │
└──────────────────────┬──────────────────────────────────────────┘
                       │ All callbacks
                       ▼
            ┌──────────────────────────┐
            │  SecondActivity          │
            │  (DetectionListener)     │
            │                          │
            │  onAccelUpdate()         │
            │  onGyroUpdate()          │
            │  onSoundUpdate()         │
            │  onGravityUpdate()       │
            │  onProximityUpdate()     │
            │  onLightUpdate()         │
            │  onActivityUpdate()      │
            │  onSpeedUpdate()         │
            │  onConfidenceUpdate()    │
            │  onStatusUpdate()        │
            │  onProbabilityUpdate()   │
            │  onAccidentConfirmed()   │
            │  → Shows UI + Alert      │
            └──────────────────────────┘
```

#### `BaseSensor.kt` (NEW — Interface)
```
interface BaseSensor {
    fun hasSensor(): Boolean
    fun start()
    fun stop()
    fun isRunning(): Boolean
}
```

All existing sensor classes would implement this interface.

#### Current Sensor Classes — Detailed Breakdown:

##### `ShakeDetector.kt` — Impact Classification (Pure Logic, No Sensor Listener)
| Aspect | Detail |
|--------|--------|
| **Input** | World-frame acceleration from `AccelProvider` via `processAccelData(wx, wy, wz, ts)` |
| **Sample Rate** | Driven by AccelProvider at ~50Hz effective |
| **Processing** | G-force conversion → jerk calculation → speed-gated thresholds → impact type classification |
| **Classification** | Dominant axis determines type: X→Forward/Rear, Y→Lateral, Z→Vertical |
| **Thresholds (g)** | Frontal: 1.2g, Side: 1.0g, Rollover: 0.8g (all speed-adjusted) |
| **Speed Gating** | <3 km/h: ×1.3 (harder), 20-50 km/h: ×0.85, >50 km/h: ×0.7 (more sensitive) |
| **Jerk Detection** | Δg/s ≥ 2.0 g/s triggers impact |
| **Buffer** | Rolling 20-sample g-force buffer; spike = (max - mean) > 1.2g |
| **Post-Impact** | 2000ms monitoring window for secondary motion confirmation |
| **Callbacks** | `onCrashDetected(type, gForce, location)`, `onImpactValues(x,y,z,gForce,type)`, `onError(msg)` |

##### `SoundProvider.kt` — Microphone dBFS Monitor
| Aspect | Detail |
|--------|--------|
| **Source** | `MediaRecorder.AudioSource.MIC` via `AudioRecord` |
| **Config** | 44100 Hz, MONO, PCM_16BIT |
| **Metric** | dBFS = 20 × log10(RMS / 32768). Range: -120 (silence) to 0 (max) |
| **Calibration** | 1-second auto-calibration window at start; baseline capped at -3.0 dBFS |
| **Polling** | 200ms intervals via `Handler.postDelayed` |
| **Threshold** | Default: -50.0 dBFS (configurable) |
| **Callbacks** | `onThresholdExceeded(db)`, `onPermissionDenied()`, `onError(exception)` |

##### `GravityProvider.kt` — Rollover Detector
| Aspect | Detail |
|--------|--------|
| **Sensor** | `TYPE_GRAVITY` at `SENSOR_DELAY_GAME` |
| **Calibration** | 20-sample averaging to establish baseline orientation |
| **Rollover** | Triggered when any axis shifts >7.0 m/s² from baseline; 3s debounce |
| **Orientation** | Classifies: FLAT_FACE_UP, FLAT_FACE_DOWN, PORTRAIT_UP/DOWN, LANDSCAPE_LEFT/RIGHT |
| **Confidence** | Upside down: +20, orientation changed: +10, normal: +0 |
| **Tilt Angle** | Calculated from gravity Y-component via arccos |
| **Callbacks** | `onGravityChange(x,y,z,orientation)`, `onRolloverDetected(from,to)` |

##### `GyroProvider.kt` — Rotation Peak Detector
| Aspect | Detail |
|--------|--------|
| **Sensor** | `TYPE_GYROSCOPE` at `SENSOR_DELAY_GAME` |
| **Metric** | Rotational velocity magnitude √(x²+y²+z²) in rad/s |
| **Threshold** | Peak ≥ 100 °/s (configurable, constructor param) |
| **Callbacks** | `onValues(x,y,z,mag)`, `onPeak(mag)`, `onError(exception)` |

##### `LightProvider.kt` — Sudden Darkness Detector
| Aspect | Detail |
|--------|--------|
| **Sensor** | `TYPE_LIGHT` at `SENSOR_DELAY_NORMAL` |
| **Calibration** | 10-sample averaging for baseline lux |
| **Darkness** | Triggered when current < 30% of baseline AND lux dropped by >50% since last reading |
| **Conditions** | Dark (<10), Dim (<50), Indoor (<300), Bright Indoor (<1000), Outdoor (<10000), Sunlight |
| **Confidence** | <20% baseline: +10, <40% baseline: +5, normal: +0 |
| **Callbacks** | `onLightChange(lux, suddenDarkness)`, `onSuddenDarkness()` |

##### `ProximityProvider.kt` — Pocket/Cover Detection
| Aspect | Detail |
|--------|--------|
| **Sensor** | `TYPE_PROXIMITY` at `SENSOR_DELAY_NORMAL` |
| **Logic** | Binary: distance < maxRange → NEAR (in pocket), else FAR (exposed) |
| **Confidence** | NEAR: +15 (more likely real accident), FAR: +0 |
| **Callback** | `onProximityChange(isNear, distance)` |

##### `BarometerProvider.kt` — Pressure Spike Detector
| Aspect | Detail |
|--------|--------|
| **Sensor** | `TYPE_PRESSURE` at `SENSOR_DELAY_FASTEST` |
| **Metric** | Atmospheric pressure (hPa); tracks delta between consecutive readings |
| **Spike Detection** | Δ ≥ 0.5 hPa within 100ms → `onPressureSpike()` (cabin pressure change = impact) |
| **Debounce** | 2000ms between spike alerts |
| **Optional** | Gracefully skipped if device has no barometer |
| **Callbacks** | `onPressureData(hPa, ts)`, `onPressureSpike(delta)` |

##### `LocationProvider.kt` — GPS Speed & Position
| Aspect | Detail |
|--------|--------|
| **Source** | `LocationManager` — GPS_PROVIDER (preferred) or NETWORK_PROVIDER fallback |
| **Updates** | Every 1000ms / 1m minimum distance |
| **Provides** | `lastSpeedMps()`, `lastLocation` (for emergency coordinates) |
| **Permissions** | `ACCESS_FINE_LOCATION` or `ACCESS_COARSE_LOCATION` |

##### `OrientationHelper.kt` — World-Frame Transform
| Aspect | Detail |
|--------|--------|
| **Sensor** | `TYPE_ROTATION_VECTOR` at `SENSOR_DELAY_GAME` |
| **Purpose** | Maintains 3×3 rotation matrix; transforms device-frame vectors to world-frame |
| **Method** | `rotateToWorld(ax, ay, az)` → FloatArray[3] (world X, Y, Z) |
| **Used By** | `ShakeDetector` to get orientation-independent impact directions |

##### `ActivityDetector.kt` — Motion State Classifier
| Aspect | Detail |
|--------|--------|
| **Sensor** | `TYPE_STEP_COUNTER` + external speed from `LocationProvider` |
| **States** | `UNKNOWN`, `STILL`, `WALKING` (>30 steps/min), `IN_VEHICLE` (>5 m/s or >1 m/s without steps) |
| **Multiplier** | IN_VEHICLE: ×1.5, STILL: ×1.0, WALKING: ×0.3 (suppresses false positives), UNKNOWN: ×1.0 |
| **Callback** | `onActivityChange(activity)` |

##### `HighPassFilter.kt` — Gravity Removal
| Aspect | Detail |
|--------|--------|
| **Algorithm** | Exponential smoothing: gravity_est = α × prev + (1-α) × raw; output = raw - gravity_est |
| **Alpha** | 0.9 (slow adaptation = better gravity estimate) |
| **Used When** | Device lacks `TYPE_LINEAR_ACCELERATION` sensor |

##### `DetectionCapabilities.kt` — Runtime Capability Check
| Aspect | Detail |
|--------|--------|
| **Checks** | 11 capabilities: Accelerometer, Gyroscope, RotationVector, LinearAcceleration, Gravity, Proximity, Light, StepCounter, Barometer, Microphone, GPS |
| **Score** | `capabilityScore()` = (available / 11) × 100% |
| **UI** | Shown as AlertDialog before starting detection; app gracefully skips missing sensors |

---

### 3. `data/` — Data Layer

#### `TrainingLogger.kt` — ML Training Data Recorder
| Aspect | Detail |
|--------|--------|
| **Format** | Append-only CSV, one file per session |
| **Location** | `Android/data/com.example.healer/files/logs/session_<timestamp>.csv` (no storage permission needed) |
| **Schema** | `timestamp, uptime_ms, acc_x, acc_y, acc_z, g_force, dbfs, gyro_x, gyro_y, gyro_z, speed_mps, activity_state, impact_type, event` |
| **Events** | `sensor` (continuous), `sound` (threshold), `gyro` (rotation), `crash` (confirmed impact) |
| **Threading** | Single-thread executor; buffered writes flushed every 32 KB |
| **Methods** | `logSensor(...)`, `logSound(...)`, `logGyro(...)`, `logEvent(...)`, `close()` |

---

### 4. `ml/` — Machine Learning Layer (NEW — Planned)

#### `AccidentClassifier.kt` — TFLite Model Runner
| Aspect | Detail |
|--------|--------|
| **Purpose** | Loads `accident_model.tflite` from assets; runs inference on feature vectors |
| **Input** | FloatArray of extracted features (from `FeatureExtractor`) |
| **Output** | `PredictionResult(label: String, confidence: Float)` — e.g., "ACCIDENT" @ 0.92 |
| **Delegate** | CPU default; NNAPI or GPU delegate optional for speed |
| **Lifecycle** | `load()` on app start, `predict(features)` per window, `close()` on shutdown |

#### `SignalProcessor.kt` — Noise Filter & Normalizer
| Aspect | Detail |
|--------|--------|
| **Purpose** | Cleans raw sensor streams before feature extraction |
| **Operations** | Low-pass filtering, outlier removal, resampling to fixed rate (e.g., 50 Hz), normalization (z-score or min-max) |
| **Input** | Raw timestamped sensor samples from `sensors/` layer |
| **Output** | Clean, uniformly-sampled FloatArray windows |

#### `FeatureExtractor.kt` — Raw Data → ML Input
| Aspect | Detail |
|--------|--------|
| **Purpose** | Converts cleaned sensor windows into feature vectors for the model |
| **Window** | 2.0s windows, 50% overlap (configurable) |
| **Time-Domain Features** | Mean, std, min, max, RMS, zero-crossing rate per axis |
| **Frequency-Domain** | FFT energy bands, dominant frequency, spectral entropy |
| **Cross Features** | Inter-axis correlation, magnitude delta, jerk statistics |
| **Context Features** | GPS speed, activity state, proximity state, light level |
| **Output** | Single FloatArray ready for `AccidentClassifier.predict()` |

#### `accident_model.tflite` — Trained Model (To Be Created)
| Aspect | Detail |
|--------|--------|
| **Architecture** | 1D-CNN or small GRU/LSTM for temporal patterns |
| **Input Shape** | `[1, window_size, num_features]` |
| **Output** | Softmax over classes: `[NORMAL, ACCIDENT, FALL, DROP]` |
| **Size Target** | <2 MB (int8 quantized) |
| **Latency Target** | <50ms on mid-range devices |

---

### 5. `service/` — Background Service Layer (NEW — Planned)

#### `DetectionService.kt` — Foreground Service
| Aspect | Detail |
|--------|--------|
| **Purpose** | Keeps sensor detection alive when app is backgrounded or screen is off |
| **Type** | Android Foreground Service with persistent notification |
| **Contains** | Own instances of all sensor providers + ML classifier |
| **Lifecycle** | Started by `SecondActivity`, survives `onPause`/`onStop`, stopped explicitly |
| **Notification** | Shows "Accident Detection Active" with current status |

---

## 🔌 Sensor Data Flow (Current — Refactored)

```
┌─────────────────────────────────────────────────────────────────┐
│                        HARDWARE SENSORS                         │
│  Accel  Gyro  Gravity  Light  Proximity  Mic  GPS  Baro        │
└──────┬──────────┬──────────┬────────┬───────┬─────────┬────┬───┘
       │          │          │        │       │         │    │
       ▼          ▼          ▼        ▼       ▼         ▼    ▼
┌──────────────────────────────────────────────────────────────────┐
│                     sensors/ (Raw Providers)                     │
│   AccelProvider  GyroProvider  GravityProvider  SoundProvider    │
│   LightProvider  ProximityProvider  BarometerProvider            │
│   LocationProvider  OrientationHelper  ActivityDetector          │
│                                                                  │
│   All implement BaseSensor interface                             │
│   Data only — no detection logic                                 │
└──────────────────────────┬──────────────────────────────────────┘
                           │ Raw callbacks + timestamps
                           ▼
┌──────────────────────────────────────────────────────────────────┐
│           detection/DetectionCoordinator (Fusion Engine)         │
│                                                                  │
│  Event Timestamps (6 channels):                                  │
│  ├── shakeDetectedTime    (from AccelProvider → ShakeDetector)   │
│  ├── soundDetectedTime    (from SoundProvider)                   │
│  ├── gyroDetectedTime     (from GyroProvider, ≥100°/s)          │
│  ├── lightDarknessTime    (from LightProvider)                   │
│  ├── rolloverTime         (from GravityProvider)                 │
│  └── barometerSpikeTime   (from BarometerProvider)               │
│                                                                  │
│  Fusion: hasPairWithinWindow(1500ms) → 2+ events correlated?    │
│  Scoring: calculateProbability() → weighted 0-100%               │
│  Suppression: isLikelyWalking() → block, ×0.3 multiplier        │
│                                                                  │
│  Output: ACCIDENT CONFIRMED → Alert + Vibrate + Log             │
└──────────────────┬─────────────────────────────────────────────┘
                   │ DetectionListener callbacks
           ┌───────┴───────┐
           ▼               ▼
    ┌──────────────┐ ┌──────────────┐
    │SecondActivity│ │  data/       │
    │(UI only)     │ │  Logger +    │
    │18+ TextViews │ │  Buffer      │
    └──────────────┘ └──────────────┘
```

---

## 🔌 Proposed Data Flow (With ML)

```
┌─────────────────────────────────────────────────────────────────┐
│                        HARDWARE SENSORS                         │
└──────┬──────────┬──────────┬────────┬───────┬─────────┬────┬───┘
       ▼          ▼          ▼        ▼       ▼         ▼    ▼
┌──────────────────────────────────────────────────────────────────┐
│                     sensors/ (Raw Providers)                     │
│   AccelProvider  GyroProvider  GravityProvider  SoundProvider    │
│   LightProvider  ProximityProvider  LocationProvider             │
│   OrientationHelper  ActivityDetector                            │
└──────────────────────────┬──────────────────────────────────────┘
                           │ Raw timestamped samples
                           ▼
┌──────────────────────────────────────────────────────────────────┐
│                     ml/SignalProcessor                           │
│   Low-pass filter → Outlier removal → Resample to 50Hz          │
│   → Z-score normalization                                        │
└──────────────────────────┬──────────────────────────────────────┘
                           │ Clean uniform windows
                           ▼
┌──────────────────────────────────────────────────────────────────┐
│                    ml/FeatureExtractor                            │
│   2s windows, 50% overlap                                        │
│   Time: mean, std, RMS, zero-cross per axis                      │
│   Freq: FFT energy, dominant freq, spectral entropy              │
│   Cross: correlation, magnitude delta, jerk stats                │
│   Context: speed, activity, proximity, light                     │
└──────────────────────────┬──────────────────────────────────────┘
                           │ Feature vector (FloatArray)
                           ▼
            ┌──────────────────────────────┐
            │    ml/AccidentClassifier      │
            │    (TFLite interpreter)       │
            │                              │
            │  Input: [1, features]        │
            │  Output: [NORMAL, ACCIDENT,  │
            │           FALL, DROP]        │
            │  + confidence score          │
            └──────────────┬───────────────┘
                           │ PredictionResult
                           ▼
┌──────────────────────────────────────────────────────────────────┐
│              detection/DetectionCoordinator                       │
│                                                                  │
│   ML prediction + Rule-based detectors (existing fusion)         │
│   Temporal voting / smoothing                                    │
│   Confidence threshold check                                     │
│   Activity-based suppression                                     │
│                                                                  │
│   Output: CONFIRMED ACCIDENT → Alert + Emergency flow            │
└──────────────────────────┬──────────────────────────────────────┘
                           │
              ┌────────────┴────────────┐
              ▼                         ▼
    ┌──────────────────┐     ┌──────────────────┐
    │   ui/ (Alert UI)  │     │  data/Logger      │
    │   Vibration       │     │  (CSV for         │
    │   Emergency call  │     │   retraining)     │
    └──────────────────┘     └──────────────────┘
```

---

## 📊 Probability Scoring Algorithm (Current)

```
Base Score:
  Impact detected (any type)     → +30
  G-force > 2.5g                 → +25
  Sound > threshold (-50 dBFS)   → +15
  Secondary motion confirmed     → +5

Sensor Boosts:
  Proximity: phone covered/pocket → +15
  Light: sudden darkness (<20%)   → +10
  Light: moderate drop (<40%)     → +5
  Gravity: upside down            → +20
  Gravity: orientation changed    → +10

Activity Multiplier:
  IN_VEHICLE   → ×1.5  (high confidence)
  STILL        → ×1.0  (normal)
  UNKNOWN      → ×1.0  (normal)
  WALKING      → ×0.3  (likely false positive — suppressed)

Final = clamp(base + boosts) × multiplier, range [0, 100]
```

---

## 📝 Training Data Schema (CSV)

```csv
timestamp,uptime_ms,acc_x,acc_y,acc_z,g_force,dbfs,gyro_x,gyro_y,gyro_z,speed_mps,impact_type,event
1708000000000,123456,0.12345,-0.23456,9.78123,1.00234,-65.43210,,,0.00000,,sensor
1708000000200,123656,,,,,,-0.01234,0.02345,-0.00567,0.00000,,gyro
1708000000400,123856,,,,,-52.12345,,,,0.00000,,sound
1708000001000,124456,1.23456,-2.34567,0.56789,2.56789,-45.00000,,,5.50000,Lateral Impact,crash
```

**Event Types:**
| Event | Source | When |
|-------|--------|------|
| `sensor` | AccelProvider → ShakeDetector | Every 50ms (continuous accelerometer) |
| `sound` | SoundProvider | Every 200ms (continuous microphone) |
| `gyro` | GyroProvider | On gyroscope change |
| `crash` | ShakeDetector (via DetectionCoordinator) | On confirmed impact detection |

---

## 🛠️ Tech Stack

| Component | Technology |
|-----------|-----------|
| **Language** | Kotlin |
| **Platform** | Android |
| **Min SDK** | API 24 (Android 7.0 Nougat) |
| **Target SDK** | API 35 |
| **Build System** | Gradle with Kotlin DSL |
| **View Binding** | Enabled |
| **Java Compat** | JVM Target 17 |
| **Analytics** | Firebase Crashlytics |
| **ML Runtime** | TensorFlow Lite (planned) |
| **ML Training** | Python + TensorFlow/PyTorch (offline) |

---

## 📱 Permissions

| Permission | Required | Purpose |
|-----------|----------|---------|
| `RECORD_AUDIO` | ✅ Yes | Sound detection (dBFS monitoring) |
| `ACCESS_FINE_LOCATION` | Optional | GPS speed for speed-gated thresholds |
| `ACCESS_COARSE_LOCATION` | Optional | Fallback location |
| `VIBRATE` | Auto | Alert feedback |
| `WAKE_LOCK` | Auto | Keep detection alive (planned) |
| `MODIFY_AUDIO_SETTINGS` | Auto | Audio configuration |
| `FOREGROUND_SERVICE` | Planned | Background detection service |

---

## 🔋 Battery & Performance Considerations

| Concern | Strategy |
|---------|----------|
| Sensor polling rate | `SENSOR_DELAY_UI` (~60Hz) for accel; `SENSOR_DELAY_GAME` for gyro/gravity; `SENSOR_DELAY_NORMAL` for light/proximity |
| Sound recording | 200ms polling intervals (not continuous streaming) |
| GPS updates | 1s / 1m minimum intervals |
| ML inference | Target <50ms per window; run on Default dispatcher |
| CSV logging | Buffered writes, flush every 32KB |
| Background | Planned foreground service with wake lock |

---

## 📅 Roadmap & Timeline

**Current Phase: Detection Maturity & ML Integration (Mar 2026 – Jun 2027)**

| Milestone | Completion | Status |
|-----------|-----------|--------|
| ✅ Core sensor wrappers (10+ sensors) | ✓ | Complete |
| ✅ BaseSensor interface abstraction | ✓ | Complete (refactored) |
| ✅ Separation of concerns (sensors/ vs detection/) | ✓ | Complete (refactored) |
| ✅ Multi-sensor fusion algorithm (6-channel, 1500ms window) | ✓ | Complete |
| ✅ Impact classification (Frontal, Lateral, Vertical, Rollover) | ✓ | Complete |
| ✅ Speed-gated thresholds (4 speed bands) | ✓ | Complete |
| ✅ Activity-based false positive suppression (WALKING ×0.3) | ✓ | Complete |
| ✅ Barometer pressure spike detection (Δ ≥ 0.5 hPa / <100ms) | ✓ | Complete |
| ✅ Training data CSV logger (world-frame normalized) | ✓ | Complete |
| ✅ Live sensor dashboard UI (18+ readouts) | ✓ | Complete |
| ✅ Runtime capability detection (11 sensors scanned) | ✓ | Complete |
| ✅ Circular sensor buffer (2s sliding window, ML-ready) | ✓ | Complete |
| ✅ DetectionCoordinator (centralized fusion engine) | ✓ | Complete |
| ✅ Auto-calibration (sound, light, gravity) | ✓ | Complete |
| ✅ Edge case handling (missing HW, permissions, debounce) | ✓ | Complete |
| ✅ Acoustic frequency analysis (FFT 4-band) | ✓ | Complete |
| ✅ Device-profile calibration (30+ devices) | ✓ | Complete |
| 🔧 ML feature extraction scaffolding | ~60% | In Progress |
| 📋 TFLite model integration | ~0% | Planned (Q4 2026) |
| 📋 Background detection service | ~0% | Planned (Q3 2026) |
| 📋 Emergency notification system | ~0% | Planned (Q3 2026) |
| 📋 Cloud backup & OTA model updates | ~0% | Planned (Q2 2027) |
| 📋 Public source code release | ~0% | Planned (Jun 2027) |

---

## 📞 Contact

**Developer:** Anbu

- **GitHub:** [@Anbu-2006](https://github.com/Anbu-2006)

Feel free to reach out for collaboration opportunities or questions about the project!

---

<p align="center">
  <i>🩺 Building safety through technology 🩺</i>
</p>

