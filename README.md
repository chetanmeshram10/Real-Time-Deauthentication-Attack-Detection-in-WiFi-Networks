# 🛡️ Real-Time Wi-Fi Deauthentication Attack Detection System

<div align="center">

![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Flask](https://img.shields.io/badge/Flask-3.0-000000?style=for-the-badge&logo=flask&logoColor=white)
![Scapy](https://img.shields.io/badge/Scapy-2.5-009C7C?style=for-the-badge)
![Platform](https://img.shields.io/badge/Platform-Ubuntu%20Linux-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Active-brightgreen?style=for-the-badge)

**A real-time Wi-Fi deauthentication attack detector with a live web dashboard, IoT device monitoring, and five independent statistical detection methods — no machine learning required.**

[Features](#-features) · [Architecture](#-architecture) · [Detection Methods](#-detection-methods) · [Installation](#-installation) · [Usage](#-usage) · [Dashboard](#-dashboard) · [Demo](#-demo)

</div>

---

## 📌 Overview

Deauthentication (deauth) attacks are a class of Wi-Fi denial-of-service attacks where an adversary sends forged IEEE 802.11 management frames to disconnect legitimate clients from an access point. These attacks are trivially launched using tools like `aireplay-ng` or `mdk4` and require no authentication.

This project implements a **real-time detection system** that:

- Passively sniffs 802.11 management frames using a monitor-mode Wi-Fi adapter
- Runs **5 independent statistical detection algorithms** simultaneously
- Tracks connected IoT devices (e.g. IP cameras) and alerts when they go offline
- Streams all events to a **live Flask + WebSocket dashboard** accessible from any browser on the network

> ⚠️ **Ethical Use Only.** This tool is designed for **defensive monitoring and academic research** on networks you own or have explicit permission to test. Unauthorized deauth attacks are illegal in most jurisdictions.

---

## ✨ Features

- 🔍 **5 Detection Methods** running in parallel — each catches different attack signatures
- 📊 **Live Dashboard** with real-time alert feed, bar charts, and severity stats
- 📷 **IoT Device Tracking** — continuously pings devices and alerts on disconnect
- 🔔 **Toast Notifications** for every new alert and device status change
- 📁 **Persistent Logging** to `logs/dashboard.log` for post-incident analysis
- 🌐 **Network-wide access** — dashboard accessible from any browser on the LAN
- ⚡ **Zero ML dependency** — pure statistical methods, runs on low-spec hardware
- 🧩 **Modular architecture** — detectors are independent and easily extensible

---

## 🏗️ Architecture

```
┌─────────────────────┐        802.11 Deauth Frames        ┌──────────────────────────┐
│   ATTACKER PC       │ ─────────────────────────────────► │   DETECTOR PC            │
│                     │                                     │                          │
│  • aireplay-ng      │                                     │  ┌──────────────────┐   │
│  • mdk4             │                                     │  │  WiFi Monitor     │   │
│  • MediaTek USB     │                                     │  │  (monitor mode)   │   │
│    adapter          │                                     │  └────────┬─────────┘   │
└─────────────────────┘                                     │           │              │
                                                            │  ┌────────▼─────────┐   │
          Wi-Fi Network (AP / Hotspot)                      │  │  5 Detectors     │   │
┌─────────────────────┐                                     │  │  running in      │   │
│   VICTIM DEVICE     │ ◄── gets disconnected               │  │  parallel        │   │
│   IP Camera / PC    │                                     │  └────────┬─────────┘   │
│   Smart Device      │                                     │           │              │
└─────────────────────┘                                     │  ┌────────▼─────────┐   │
                                                            │  │  Flask + SocketIO│   │
                                                            │  │  Dashboard :5000 │   │
                                                            │  └──────────────────┘   │
                                                            └──────────────────────────┘
                                                                        ▲
                                                              Browser on any device
                                                              http://<detector-ip>:5000
```

---

## 🔬 Detection Methods

All five methods run simultaneously. Any single method triggering fires an alert.

| # | Method | How It Works | Accuracy | Severity |
|---|--------|-------------|----------|----------|
| 1 | **Threshold Detection** | Counts deauth frames in a sliding time window; alerts when count exceeds configurable limit | 85% | HIGH |
| 2 | **Broadcast Detection** | Flags any deauth frame sent to `ff:ff:ff:ff:ff:ff` — legitimate APs never do this | 90% | CRITICAL |
| 3 | **Rate Anomaly Detection** | Learns baseline deauth rate, then uses Z-score to flag statistically abnormal spikes | 75% | MEDIUM |
| 4 | **Rapid Sequence Detection** | Measures inter-frame timing and jitter; identifies machine-generated burst patterns | 80% | HIGH |
| 5 | **Reason Code Analysis** | Validates IEEE 802.11 reason codes; flags invalid, reserved, or attack-common codes | 88% | HIGH/MEDIUM |

---

## 📁 Project Structure

```
deauth_dashboard/
├── app.py                          # Flask entry point + SocketIO server
├── requirements.txt
├── setup.sh                        # One-shot Ubuntu setup script
│
├── core/
│   ├── monitor.py                  # Packet capture + detector orchestration
│   ├── alert_store.py              # Thread-safe in-memory alert storage
│   ├── device_tracker.py           # IoT device ping monitor
│   └── detectors/
│       ├── threshold.py            # Method 1 — Frame count threshold
│       ├── broadcast.py            # Method 2 — Broadcast MAC detection
│       ├── rate_anomaly.py         # Method 3 — Baseline rate Z-score
│       ├── rapid_seq.py            # Method 4 — Timing pattern analysis
│       └── reason_code.py          # Method 5 — IEEE 802.11 reason codes
│
├── templates/
│   └── dashboard.html              # Full real-time dashboard (single file)
│
├── utils/
│   └── logger.py                   # Logging utility
│
└── logs/
    └── dashboard.log               # Auto-generated detection log
```

---

## ⚙️ Installation

### Prerequisites

- Ubuntu 20.04+ (or any Debian-based Linux)
- Python 3.10+
- A Wi-Fi adapter that supports **monitor mode** (e.g. MediaTek MT7612U, Alfa AWUS036ACH)
- `aircrack-ng` suite

### Step 1 — Clone the repository

```bash
git clone https://github.com/<your-username>/deauth-detection-dashboard.git
cd deauth-detection-dashboard
```

### Step 2 — Run the setup script

```bash
sudo bash setup.sh wlan0 192.168.1.100
#                  ^^^^^  ^^^^^^^^^^^^^
#                  your   webcam/IoT IP (optional)
#                  interface
```

This will:
- Install all system and Python dependencies
- Enable monitor mode on your adapter
- Print your exact start command

### Step 3 — Manual install (alternative)

```bash
sudo apt update
sudo apt install -y aircrack-ng iw wireless-tools tcpdump iputils-ping
pip3 install flask flask-socketio scapy eventlet --break-system-packages
```

---

## 🚀 Usage

### Detector PC

**Step 1 — Enable monitor mode**
```bash
sudo airmon-ng check kill
sudo airmon-ng start wlan0
# Interface is now: wlan0mon
```

**Step 2 — Lock to target channel**
```bash
sudo iw dev wlan0mon set channel 6
```

**Step 3 — Start the dashboard**
```bash
sudo python3 app.py -i wlan0mon --webcam 192.168.1.100
```

With all options:
```bash
sudo python3 app.py \
  -i wlan0mon \
  --webcam 192.168.1.100 \
  --devices 192.168.1.101 192.168.1.102 \
  --threshold 8 \
  --window 5 \
  --baseline-period 15 \
  --port 5000 \
  -v
```

**Step 4 — Open the dashboard**

From any browser on the same network:
```
http://<detector-pc-ip>:5000
```

Find your IP with: `hostname -I`

---

### Attacker PC (for testing only — your own network)

```bash
# Enable monitor mode on MediaTek adapter
sudo airmon-ng check kill
sudo airmon-ng start wlan1

# Discover target AP
sudo airodump-ng wlan1mon

# Lock to target channel
sudo airodump-ng --bssid AA:BB:CC:DD:EE:FF --channel 6 wlan1mon

# Launch broadcast deauth attack
sudo aireplay-ng --deauth 0 -a AA:BB:CC:DD:EE:FF wlan1mon

# Targeted attack against specific client
sudo aireplay-ng --deauth 0 -a AA:BB:CC:DD:EE:FF -c CLIENT:MAC wlan1mon
```

---

## CLI Reference

| Argument | Default | Description |
|----------|---------|-------------|
| `-i`, `--interface` | required | Monitor-mode interface (e.g. `wlan0mon`) |
| `--webcam` | None | IP address of webcam to track |
| `--devices` | None | Additional device IPs to monitor |
| `--threshold` | `10` | Deauth frames per window to trigger alert |
| `--window` | `5` | Sliding window size in seconds |
| `--baseline-period` | `15` | Seconds to learn baseline before activating |
| `--port` | `5000` | Flask server port |
| `-v`, `--verbose` | False | Enable debug logging |

---

## 📊 Dashboard

The web dashboard provides:

- **Stat Cards** — total alerts, critical / high / medium counts updated live
- **Bar Chart** — alert count per detection method
- **Device Panel** — real-time online/offline status of all tracked IoT devices with disconnect counter
- **Alert Feed** — scrollable table of every alert with timestamp, method, severity, source MAC, and detail
- **Toast Notifications** — popup for every new alert and device status change
- **Clear Button** — reset all alerts during a session

All updates are pushed via **WebSocket (SocketIO)** — no page refresh required.

---

## 🔧 Tuning Guide

| Symptom | Fix |
|---------|-----|
| Too many false positives | Raise `--threshold` or increase `--baseline-period` |
| Missing attacks | Lower `--threshold` to `3–5`, shorten `--window` |
| Rate anomaly too noisy | Increase `SIGMA_MULTIPLIER` in `rate_anomaly.py` (default `3.0`) |
| Reason code false positives | Edit `LEGITIMATE_REASONS` set in `reason_code.py` |
| Detector sees no frames | Verify both machines are on the **same channel** |
| Device tracker flapping | Increase `OFFLINE_AFTER` in `device_tracker.py` (default `2`) |

---

## 🛠️ Troubleshooting

| Problem | Solution |
|---------|----------|
| `Operation not permitted` | Run with `sudo` |
| `No such device` | Check interface name with `iw dev` |
| Monitor mode won't enable | Run `sudo airmon-ng check kill` first |
| Target AP not in airodump | Force phone hotspot to **2.4 GHz**; try `--band abg` flag |
| Dashboard not loading | Check `hostname -I` for correct IP; ensure port 5000 is not firewalled |
| Webcam not detected | Verify webcam IP with `sudo arp-scan --localnet` |

---

## 🧰 Tech Stack

| Component | Technology |
|-----------|-----------|
| Packet capture | [Scapy](https://scapy.net/) |
| Web framework | [Flask](https://flask.palletsprojects.com/) |
| Real-time events | [Flask-SocketIO](https://flask-socketio.readthedocs.io/) + [Socket.IO](https://socket.io/) |
| Charts | [Chart.js](https://www.chartjs.org/) |
| Attack tooling (testing) | [Aircrack-ng](https://www.aircrack-ng.org/) suite |

---

## 🤝 Contributing

Contributions are welcome! To add a new detection method:

1. Create `core/detectors/your_method.py`
2. Implement `analyze(frame, window_frames) -> {"alert": bool, "severity": str, "detail": str}`
3. Optionally implement `record_baseline(frame_info)` for baseline-aware detectors
4. Register it in `core/monitor.py` under `self.detectors`

---

## 📄 License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.

---

## ⚠️ Disclaimer

This tool is intended **strictly for educational purposes and authorized network security testing**. The authors take no responsibility for any misuse. Always obtain explicit written permission before testing on any network you do not own.

---

## 👨‍💻 Author

## **Chetan Meshram**

**Mtech IT IIITA Student | Blockchain & Cryptography Enthusiast**

🔗 GitHub: https://github.com/chetanmeshram10

---

## ⭐ Support
If you found this project useful:
- ⭐ Star the repository
- 🍴 Fork it
- 📢 Share
