# 🚁 Drone Building Troubleshooting Guide

[![Made with ❤️ by The Drone Community](https://img.shields.io/badge/Made%20with%20%E2%9D%A4%EF%B8%8F%20by-The%20Drone%20Community-blue)](https://github.com/your-org/drone-troubleshooting-guide)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](http://makeapullrequest.com)

> **50 Most Common Problems & Solutions for DIY Drone Builders**  
> From receiver binding to PID tuning — everything you need to get your drone in the air.

---

## 📖 About This Guide

This comprehensive troubleshooting guide was created for **The Drone Community** workshops to help beginners and intermediate builders solve the most frustrating problems they encounter during drone builds.

Each problem includes:
- ✅ **Symptoms** — What you'll actually see/experience
- 🔍 **Common Causes** — Why it's happening
- 🛠️ **Step-by-Step Solution** — Exactly how to fix it
- 💻 **Commands/Settings** — Copy-paste ready CLI commands
- 💡 **Pro Tips** — Hard-won wisdom from experienced builders
- ⚠️ **Warnings** — Things that can damage your equipment

---

## 📁 Repository Structure

```
drone-troubleshooting-guide/
├── README.md                          # You are here
├── CONTRIBUTING.md                    # How to contribute
├── LICENSE                            # MIT License
│
├── guides/                            # All troubleshooting guides
│   ├── 01-receiver-radio-issues.md    # Problems 1-5
│   ├── 02-flight-controller.md        # Problems 6-10
│   ├── 03-esc-motor-issues.md         # Problems 11-15
│   ├── 04-gps-compass.md              # Problems 16-20
│   ├── 05-companion-computer.md       # Problems 21-25
│   ├── 06-power-system.md             # Problems 26-30
│   ├── 07-telemetry-ground-station.md # Problems 31-35
│   ├── 08-sensor-calibration.md       # Problems 36-40
│   ├── 09-arming-preflight.md         # Problems 41-45
│   └── 10-flight-tuning.md            # Problems 46-50
│
└── assets/                            # Diagrams and images
    └── wiring-diagrams/               # Wiring reference images
```

---

## 📋 Quick Problem Index

### Category 1: Receiver & Radio Issues
| # | Problem | Guide |
|---|---------|-------|
| 1 | ELRS/CRSF Receiver Not Detected | [01-receiver-radio-issues.md](guides/01-receiver-radio-issues.md#problem-1-elrscrsf-receiver-not-detected) |
| 2 | ExpressLRS Binding Failure | [01-receiver-radio-issues.md](guides/01-receiver-radio-issues.md#problem-2-expresslrs-binding-failure) |
| 3 | Stick Inputs Reversed/Wrong | [01-receiver-radio-issues.md](guides/01-receiver-radio-issues.md#problem-3-stick-inputs-reversedwrong-channel-order) |
| 4 | Receiver Loses Connection Mid-Flight | [01-receiver-radio-issues.md](guides/01-receiver-radio-issues.md#problem-4-receiver-loses-connection-mid-flight) |
| 5 | SBUS Not Working on F4 Flight Controller | [01-receiver-radio-issues.md](guides/01-receiver-radio-issues.md#problem-5-sbus-not-working-on-f4-flight-controller) |

### Category 2: Flight Controller & Firmware
| # | Problem | Guide |
|---|---------|-------|
| 6 | FC Not Detected by Computer | [02-flight-controller.md](guides/02-flight-controller.md#problem-6-flight-controller-not-detected-by-computer) |
| 7 | Cannot Flash Firmware / DFU Mode Issues | [02-flight-controller.md](guides/02-flight-controller.md#problem-7-cannot-flash-firmware--dfu-mode-issues) |
| 8 | "No Gyro Detected" After Flashing | [02-flight-controller.md](guides/02-flight-controller.md#problem-8-no-gyro-detected-after-flashing) |
| 9 | CLI Settings Not Saving | [02-flight-controller.md](guides/02-flight-controller.md#problem-9-cli-settings-not-saving) |
| 10 | Random Reboots/Freezes in Flight | [02-flight-controller.md](guides/02-flight-controller.md#problem-10-random-rebootsfreezes) |

### Category 3: ESC & Motor Issues
| # | Problem | Guide |
|---|---------|-------|
| 11 | Motors Not Spinning in Betaflight | [03-esc-motor-issues.md](guides/03-esc-motor-issues.md#problem-11-motors-not-spinning-in-betaflight) |
| 12 | Wrong Motor Direction/Order | [03-esc-motor-issues.md](guides/03-esc-motor-issues.md#problem-12-wrong-motor-directionorder) |
| 13 | BLHeli Won't Connect to ESCs | [03-esc-motor-issues.md](guides/03-esc-motor-issues.md#problem-13-blheli-wont-connect-to-escs) |
| 14 | ESC Desync / Motor Stuttering | [03-esc-motor-issues.md](guides/03-esc-motor-issues.md#problem-14-esc-desync--motor-stuttering-in-flight) |
| 15 | Motor Grinding/Stuttering on Startup | [03-esc-motor-issues.md](guides/03-esc-motor-issues.md#problem-15-motor-grindingstuttering-on-startup) |

### Category 4: GPS & Compass Issues
| # | Problem | Guide |
|---|---------|-------|
| 16 | GPS Not Getting Lock | [04-gps-compass.md](guides/04-gps-compass.md#problem-16-gps-not-getting-satellite-lock) |
| 17 | Compass Calibration Always Fails | [04-gps-compass.md](guides/04-gps-compass.md#problem-17-compass-calibration-always-fails) |
| 18 | Toilet Bowl Effect in Loiter | [04-gps-compass.md](guides/04-gps-compass.md#problem-18-toilet-bowl-effect-in-loiter-mode) |
| 19 | GPS Module Not Detected | [04-gps-compass.md](guides/04-gps-compass.md#problem-19-gps-module-not-detected-by-flight-controller) |
| 20 | GPS Glitches / Position Jumps | [04-gps-compass.md](guides/04-gps-compass.md#problem-20-gps-glitches--position-jumps) |

### Category 5: Companion Computer
| # | Problem | Guide |
|---|---------|-------|
| 21 | Raspberry Pi Can't Communicate with FC | [05-companion-computer.md](guides/05-companion-computer.md#problem-21-raspberry-pi-cant-communicate-with-flight-controller) |
| 22 | Permission Denied on Serial Port | [05-companion-computer.md](guides/05-companion-computer.md#problem-22-permission-denied-on-serial-port-devttyacm0) |
| 23 | MAVLink Connection Unstable | [05-companion-computer.md](guides/05-companion-computer.md#problem-23-mavlink-connection-unstable--high-packet-loss) |
| 24 | Dronekit Commands Not Working | [05-companion-computer.md](guides/05-companion-computer.md#problem-24-dronekit-commands-ignored-by-flight-controller) |
| 25 | Camera Feed Not Working | [05-companion-computer.md](guides/05-companion-computer.md#problem-25-camera-feed-not-workinglagging) |

### Category 6: Power System
| # | Problem | Guide |
|---|---------|-------|
| 26 | Magic Smoke on Power-Up | [06-power-system.md](guides/06-power-system.md#problem-26-magic-smoke-on-first-power-up) |
| 27 | FC Brownout Under Load | [06-power-system.md](guides/06-power-system.md#problem-27-flight-controller-brownout-under-throttle) |
| 28 | Battery Connector Overheating | [06-power-system.md](guides/06-power-system.md#problem-28-battery-connector-getting-hot) |
| 29 | Battery Failsafe Triggers Wrong | [06-power-system.md](guides/06-power-system.md#problem-29-battery-failsafe-triggers-at-wrong-voltage) |
| 30 | Video Noise/Lines When Throttling | [06-power-system.md](guides/06-power-system.md#problem-30-video-noiselines-when-motors-spin) |

### Category 7: Telemetry & Ground Station
| # | Problem | Guide |
|---|---------|-------|
| 31 | "No Heartbeat Packets Received" | [07-telemetry-ground-station.md](guides/07-telemetry-ground-station.md#problem-31-no-heartbeat-packets-received) |
| 32 | Short Telemetry Range | [07-telemetry-ground-station.md](guides/07-telemetry-ground-station.md#problem-32-telemetry-range-very-short) |
| 33 | Parameters Won't Download | [07-telemetry-ground-station.md](guides/07-telemetry-ground-station.md#problem-33-parameters-wont-download--timeout) |
| 34 | OSD Elements Missing | [07-telemetry-ground-station.md](guides/07-telemetry-ground-station.md#problem-34-osd-elements-missing-or-wrong) |
| 35 | QGroundControl Won't Connect | [07-telemetry-ground-station.md](guides/07-telemetry-ground-station.md#problem-35-qgroundcontrol-wont-connect) |

### Category 8: Sensor Calibration
| # | Problem | Guide |
|---|---------|-------|
| 36 | Accelerometer Calibration Fails | [08-sensor-calibration.md](guides/08-sensor-calibration.md#problem-36-accelerometer-calibration-fails) |
| 37 | Gyro Calibration Fails | [08-sensor-calibration.md](guides/08-sensor-calibration.md#problem-37-gyro-calibration-fails-at-boot) |
| 38 | Drone Not Level After Calibration | [08-sensor-calibration.md](guides/08-sensor-calibration.md#problem-38-drone-not-level-after-calibration) |
| 39 | Barometer Altitude Wrong | [08-sensor-calibration.md](guides/08-sensor-calibration.md#problem-39-barometer-altitude-readings-wrong) |
| 40 | Sensor Orientation Wrong | [08-sensor-calibration.md](guides/08-sensor-calibration.md#problem-40-sensor-orientation-mismatch) |

### Category 9: Arming & Pre-Flight
| # | Problem | Guide |
|---|---------|-------|
| 41 | Multiple Pre-Arm Errors | [09-arming-preflight.md](guides/09-arming-preflight.md#problem-41-multiple-pre-arm-check-failures) |
| 42 | RXLOSS Flag Won't Clear | [09-arming-preflight.md](guides/09-arming-preflight.md#problem-42-rxloss-flag-wont-clear) |
| 43 | "Throttle Not at Zero" Error | [09-arming-preflight.md](guides/09-arming-preflight.md#problem-43-throttle-not-at-zero-prevents-arming) |
| 44 | Arm Switch Not Working | [09-arming-preflight.md](guides/09-arming-preflight.md#problem-44-arm-switch-not-working) |
| 45 | Hardware Safety Switch Issues | [09-arming-preflight.md](guides/09-arming-preflight.md#problem-45-hardware-safety-switch-issues-ardupilot) |

### Category 10: Flight Problems & Tuning
| # | Problem | Guide |
|---|---------|-------|
| 46 | Drone Flips on Takeoff | [10-flight-tuning.md](guides/10-flight-tuning.md#problem-46-drone-flips-on-takeoff) |
| 47 | Oscillation/Vibration in Flight | [10-flight-tuning.md](guides/10-flight-tuning.md#problem-47-oscillationvibration-in-flight) |
| 48 | Drone Drifts in Loiter/Position Hold | [10-flight-tuning.md](guides/10-flight-tuning.md#problem-48-drone-drifts-in-loiter-mode) |
| 49 | Yaw Spin / Yaw Drift | [10-flight-tuning.md](guides/10-flight-tuning.md#problem-49-yaw-spin--toilet-bowl) |
| 50 | Poor Flight Performance / Sluggish | [10-flight-tuning.md](guides/10-flight-tuning.md#problem-50-poor-flight-performance--sluggish-response) |

---

## 🚀 How to Use This Guide

### Quick Search
1. **Know your problem?** Use the index above to jump directly to it
2. **Not sure?** Browse by category in the `/guides` folder
3. **Need everything?** Clone the repo and search locally

### During a Build
```bash
# Clone for offline access
git clone https://github.com/your-org/drone-troubleshooting-guide.git

# Search for keywords
grep -ri "motor spin" guides/
```

### At Workshops
Print or display specific guides relevant to the session. Each markdown file is designed to be readable on its own.

---

## 🛠️ Covered Hardware & Software

### Flight Controllers
- Pixhawk 2.4.8 / Pixhawk 4
- F4/F7/H7 Betaflight boards
- CUAV V5+, Holybro, Matek

### Firmware
- **Betaflight** 4.3+
- **ArduPilot** Copter 4.x
- **PX4** 1.13+
- **BLHeli_S** / **BLHeli_32**

### Receivers
- ExpressLRS (ELRS)
- TBS Crossfire (CRSF)
- FrSky (SBUS, F.Port)
- FlySky (iBUS)

### Ground Stations
- Mission Planner
- QGroundControl
- Betaflight Configurator
- BLHeli Configurator

---

## 🤝 Contributing

Found a problem we missed? Have a better solution? PRs are welcome!

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

### Quick Contribution
1. Fork the repo
2. Add your problem to the appropriate guide
3. Follow the existing format
4. Submit a PR

---

## 📚 Additional Resources

- [Oscar Liang's FPV Blog](https://oscarliang.com/) — Detailed FPV tutorials
- [ArduPilot Documentation](https://ardupilot.org/copter/) — Official ArduPilot docs
- [Betaflight Wiki](https://betaflight.com/docs/wiki) — Betaflight reference
- [ExpressLRS Docs](https://www.expresslrs.org/) — ELRS setup guides
- [Joshua Bardwell YouTube](https://www.youtube.com/@JoshuaBardwell) — Video troubleshooting

---

## 📝 License

This project is licensed under the MIT License — see [LICENSE](LICENSE) for details.

---

## 🙏 Acknowledgments

Created for **[The Drone Community](https://github.com/thedronecommunity)** workshops in Pune, Bangalore, and across India.

Special thanks to:
- The ArduPilot and Betaflight communities
- Oscar Liang for countless tutorials
- Every builder who shared their problems and solutions

---

<p align="center">
  <strong>Happy Building! 🛠️🚁</strong><br>
  <em>If this helped you, consider starring ⭐ the repo!</em>
</p># drone-troubleshooting-guide
