# 🩺 Healer App

![Project Status](https://img.shields.io/badge/Status-In%20Development%20(80%25)-yellow)
![License](https://img.shields.io/badge/License-Proprietary-red)

An intelligent accident detection application for Android that leverages multiple sensors to detect potential emergencies and provide timely assistance.

---

## 📋 Project Status

**Current Progress: 80% Complete**

This project is actively under development with core detection features implemented and tested.

---

## ✨ Key Features

| Feature | Description | Status |
|---------|-------------|--------|
| 🔊 **Sound Detection** | Monitors ambient sound levels to detect loud crashes or impact sounds | ✅ Implemented |
| 📳 **Shake Detection** | Uses accelerometer to detect sudden movements or impacts | ✅ Implemented |
| 🌍 **Gravity Sensor** | Monitors device orientation and detects falls or sudden tilts | ✅ Implemented |
| 📍 **Location Provider** | Tracks GPS location for emergency services coordination | ✅ Implemented |
| 🔄 **Gyroscope Detection** | Analyzes rotational movement for accident pattern recognition | ✅ Implemented |
| 💡 **Light Sensor** | Monitors ambient light changes for environmental awareness | ✅ Implemented |
| 📱 **Proximity Sensor** | Detects nearby objects and device positioning | ✅ Implemented |
| 🧭 **Orientation Helper** | Provides comprehensive device orientation data | ✅ Implemented |

---

## 🔒 Source Code Availability

> **⚠️ IMPORTANT NOTICE**
>
> **This project is currently Closed Source for security reasons.**
>
> The source code will be released in **Q2 2026**.
>
> This repository contains **documentation and architecture designs only**.
>
> We appreciate your understanding as we ensure the security and stability of the application before making the codebase publicly available.

---

## 🏗️ Architecture Overview

The Healer App follows a modular architecture with dedicated detection components:

- **MainActivity** - Entry point and navigation hub
- **SecondActivity** - Main accident detection interface
- **Detection Module** - Contains all sensor detection classes
  - ActivityDetector
  - ShakeDetector
  - SoundDetector
  - GravitySensor
  - GyroDetector
  - LightSensor
  - ProximitySensor
  - LocationProvider
  - OrientationHelper

For detailed architecture documentation, see the `/docs` folder.

---

## 📁 Repository Structure

```
healer_public_export/
├── README.md              # This file
├── LICENSE                # Proprietary license notice
├── docs/                  # Architecture and design documentation
│   └── architecture_overview.txt
└── assets/                # Screenshots and media (coming soon)
```

---

## 🛠️ Tech Stack

- **Language:** Kotlin
- **Platform:** Android
- **Min SDK:** Android 7.0 (API 24)
- **Build System:** Gradle with Kotlin DSL

---

## 📅 Roadmap

- [x] Core sensor detection implementation
- [x] Multi-sensor fusion algorithm
- [ ] Emergency contact notification system
- [ ] Cloud backup integration
- [ ] Public source code release (Q2 2026)

---

## 📞 Contact

**Developer:** Anbu

- **GitHub:** [@Anbu-2006](https://github.com/Anbu-2006)

Feel free to reach out for collaboration opportunities or questions about the project!

---

<p align="center">
  <i>Building safety through technology</i>
</p>

