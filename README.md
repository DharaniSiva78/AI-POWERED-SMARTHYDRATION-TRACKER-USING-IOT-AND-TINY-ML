<div align="center">

# 💧 AI-Powered Smart Hydration Tracker using IoT and TinyML

**A cloud-connected, on-device-intelligent smart bottle system that detects drinking events using sensor fusion and helps prevent dehydration.**


</div>

---

## 📑 Table of Contents

- [Overview](#-overview)
- [Features](#-features)
- [System Architecture](#-system-architecture)
- [Workflow Diagram](#-workflow-diagram)
- [Data Flow Sequence](#-data-flow-sequence)
- [Sensor Fusion Logic](#-sensor-fusion-logic)
- [TinyML Pipeline](#-tinyml-pipeline)
- [Tech Stack](#-tech-stack)
- [Hardware Requirements](#-hardware-requirements)
- [Project Structure](#-project-structure)
- [Getting Started](#-getting-started)
- [Screenshots](#-screenshots)
- [Future Enhancements](#-future-enhancements)
- [Author](#-author)

---

## 📖 Overview

Dehydration often goes unnoticed in daily life. This project presents a **cloud-based intelligent hydration system** that:

- Detects **drinking events** by fusing data from a **load cell** and a **tilt sensor**
- Runs **TinyML inference on the ESP32** to analyze user hydration behavior at the edge
- Syncs data to the **cloud** in real time for storage and visualization
- Sends **proactive alerts** when the user is falling behind on hydration goals

---

## ✨ Features

| Feature | Description |
|---|---|
| 💧 Real-time intake monitoring | Continuously tracks water level and consumption |
| 🎯 Sensor-fusion detection | Combines weight and tilt data to confirm genuine drinking events |
| 🧠 TinyML analytics | On-device behavior analysis with low latency and no constant cloud dependency |
| 🔔 Intelligent alerts | Reminders triggered when intake falls behind the expected pattern |
| ☁️ Cloud sync | Stores and synchronizes hydration data via Firebase / AWS IoT |
| 📊 Web dashboard | Visualizes daily and weekly hydration trends |
| 🔋 Low-power design | Battery-powered ESP32 with sleep modes |

---

## 🏗️ System Architecture

The system follows a layered IoT architecture: **Sensing → Edge Intelligence → Connectivity → Cloud → Application**.

```mermaid
flowchart TB
    subgraph L1["🔌 Sensing Layer"]
        A1["⚖️ Load Cell + HX711<br/>(Water weight)"]
        A2["📐 Tilt Sensor<br/>(Bottle orientation)"]
        A3["🌡️ Temperature Sensor<br/>(Optional)"]
    end

    subgraph L2["🧠 Edge Intelligence Layer - ESP32"]
        B1["Signal Acquisition<br/>& Noise Filtering"]
        B2["Sensor Fusion<br/>Drink Event Detection"]
        B3["Feature Extraction"]
        B4["TinyML Model<br/>(TensorFlow Lite Micro)"]
        B5["Alert Decision Logic"]
    end

    subgraph L3["📡 Connectivity Layer"]
        C1["Wi-Fi"]
        C2["MQTT / HTTP"]
    end

    subgraph L4["☁️ Cloud Layer"]
        D1["Firebase / AWS IoT Core"]
        D2[("Cloud Database")]
        D3["Analytics & Notification Service"]
    end

    subgraph L5["📱 Application Layer"]
        E1["Web Dashboard"]
        E2["Push / Alert Notifications"]
        E3["User"]
    end

    A1 --> B1
    A2 --> B1
    A3 --> B1
    B1 --> B2
    B2 --> B3
    B3 --> B4
    B4 --> B5
    B2 --> C1
    B5 --> C1
    C1 --> C2
    C2 --> D1
    D1 --> D2
    D2 --> D3
    D2 --> E1
    D3 --> E2
    E1 --> E3
    E2 --> E3

    classDef sensing fill:#e3f2fd,stroke:#1976d2,color:#0d47a1
    classDef edge fill:#fff3e0,stroke:#f57c00,color:#e65100
    classDef conn fill:#f3e5f5,stroke:#8e24aa,color:#4a148c
    classDef cloud fill:#e8f5e9,stroke:#388e3c,color:#1b5e20
    classDef app fill:#fce4ec,stroke:#c2185b,color:#880e4f
    class A1,A2,A3 sensing
    class B1,B2,B3,B4,B5 edge
    class C1,C2 conn
    class D1,D2,D3 cloud
    class E1,E2,E3 app
```

### Layer Summary

| Layer | Components | Responsibility |
|---|---|---|
| **Sensing** | Load cell, HX711, tilt sensor, temperature sensor | Capture raw physical signals |
| **Edge Intelligence** | ESP32 + TensorFlow Lite Micro | Filter, fuse, classify, and decide locally |
| **Connectivity** | Wi-Fi, MQTT / HTTP | Reliable transmission to the cloud |
| **Cloud** | Firebase / AWS IoT | Storage, analytics, notifications |
| **Application** | Web dashboard, alerts | Visualization and user feedback |

---

## 🔄 Workflow Diagram

End-to-end operation of the hydration tracker, from sensing to alerting:

```mermaid
flowchart TD
    START([▶️ Power ON]) --> INIT[Initialize ESP32, Sensors<br/>& Wi-Fi]
    INIT --> CAL[Calibrate Load Cell<br/>Tare Bottle Weight]
    CAL --> READ[Read Weight + Tilt Data]
    READ --> FILTER[Filter Noise<br/>Moving Average / Debounce]
    FILTER --> TILT{Bottle tilted<br/>beyond threshold?}

    TILT -- No --> IDLE[Idle / Low-Power Mode]
    IDLE --> READ

    TILT -- Yes --> LIFT[Track lift → drink → place-back cycle]
    LIFT --> DELTA{Weight decreased<br/>after bottle placed back?}

    DELTA -- No --> DISCARD[Ignore false trigger<br/>e.g. bottle moved / shaken]
    DISCARD --> READ

    DELTA -- Yes --> EVENT[✅ Valid Drinking Event<br/>Intake = Weight Δ]
    EVENT --> FEATURES[Extract Features<br/>volume, timing, frequency, interval]
    FEATURES --> ML[🧠 TinyML Inference<br/>Hydration Behavior Analysis]
    ML --> STATUS{Hydration<br/>status?}

    STATUS -- On Track --> LOG[Log Event]
    STATUS -- Falling Behind --> ALERT[🔔 Trigger Reminder<br/>LED / Buzzer / Notification]
    STATUS -- Dehydration Risk --> URGENT[🚨 Urgent Alert]

    LOG --> SYNC
    ALERT --> SYNC
    URGENT --> SYNC

    SYNC[☁️ Publish Data via MQTT / HTTP] --> CLOUD[(Cloud Database)]
    CLOUD --> DASH[📊 Web Dashboard Update]
    DASH --> READ

    classDef start fill:#c8e6c9,stroke:#2e7d32,color:#1b5e20
    classDef decision fill:#fff9c4,stroke:#f9a825,color:#6d4c00
    classDef ml fill:#ffe0b2,stroke:#ef6c00,color:#bf360c
    classDef alert fill:#ffcdd2,stroke:#c62828,color:#b71c1c
    classDef cloud fill:#bbdefb,stroke:#1565c0,color:#0d47a1
    class START start
    class TILT,DELTA,STATUS decision
    class ML,FEATURES ml
    class ALERT,URGENT alert
    class SYNC,CLOUD,DASH cloud
```

---

## 🔁 Data Flow Sequence

```mermaid
sequenceDiagram
    autonumber
    actor User
    participant Bottle as 🍶 Smart Bottle (Sensors)
    participant ESP as 🧠 ESP32 + TinyML
    participant Cloud as ☁️ Cloud (Firebase / AWS IoT)
    participant Dash as 📊 Web Dashboard

    User->>Bottle: Lifts bottle and drinks
    Bottle->>ESP: Tilt + weight readings
    ESP->>ESP: Filter and fuse sensor data
    ESP->>ESP: Detect drinking event (Δ weight)
    ESP->>ESP: TinyML inference on hydration pattern
    alt Intake behind schedule
        ESP-->>User: Local alert (LED / buzzer)
    end
    ESP->>Cloud: Publish event + prediction (MQTT/HTTP)
    Cloud->>Cloud: Store and aggregate data
    Cloud-->>User: Push notification / reminder
    Dash->>Cloud: Request hydration history
    Cloud-->>Dash: Daily / weekly intake data
    Dash-->>User: Charts and hydration insights
```

---

## 🎯 Sensor Fusion Logic

Neither sensor alone is reliable. Fusing both reduces false detections:

| Load Cell (Weight) | Tilt Sensor | Interpretation |
|---|---|---|
| Stable | Upright | Idle |
| Fluctuating | Tilted | Bottle being lifted or drinking in progress |
| Decreased after settling | Upright again | ✅ **Confirmed drinking event** |
| Unchanged after settling | Upright again | ❌ False trigger (moved or shaken) |
| Increased after settling | Upright again | 🔄 Bottle refill |

```mermaid
stateDiagram-v2
    [*] --> Idle
    Idle --> Lifted: Tilt detected
    Lifted --> Drinking: Sustained tilt angle
    Drinking --> PlacedBack: Returned upright
    PlacedBack --> Confirmed: Weight decreased
    PlacedBack --> Refill: Weight increased
    PlacedBack --> FalseTrigger: No weight change
    Confirmed --> Idle: Log intake
    Refill --> Idle: Update baseline
    FalseTrigger --> Idle: Discard
```

---

## 🧠 TinyML Pipeline

```mermaid
flowchart LR
    A["📥 Collect sensor data"] --> B["🧹 Preprocess / Label"]
    B --> C["🏋️ Train model<br/>(Python + TensorFlow)"]
    C --> D["📉 Quantize<br/>(TFLite int8)"]
    D --> E["📦 Convert to C array<br/>(model.h)"]
    E --> F["⚡ Deploy on ESP32<br/>(TFLite Micro)"]
    F --> G["🔮 On-device inference"]
    G --> H["📤 Prediction + alerts"]
```

**Why TinyML?**
- ⚡ Low latency: decisions are made on the device
- 🔋 Lower power use, since less data is transmitted
- 🔒 Better privacy: raw sensor data stays local
- 📶 Works even when the network is temporarily unavailable

---

## 🛠️ Tech Stack

### Hardware
- ESP32 Microcontroller
- Load Cell Sensor + HX711 Amplifier
- Tilt Sensor
- Temperature Sensor (Optional)
- Li-ion Battery Module

### Software & Tools
- Arduino IDE
- TinyML / TensorFlow Lite
- Firebase / AWS IoT
- Python
- MQTT / HTTP Protocols
- Git & GitHub

---

## 🔧 Hardware Requirements

| Component | Purpose |
|---|---|
| ESP32 Dev Board | Main controller with Wi-Fi and edge inference |
| Load Cell + HX711 | Measures water weight accurately |
| Tilt Sensor | Detects bottle orientation |
| Temperature Sensor *(optional)* | Adjusts hydration targets for ambient conditions |
| Li-ion Battery + Charger Module | Portable power supply |

### Suggested Pin Connections *(adjust to your wiring)*

| Module | Pin | ESP32 GPIO |
|---|---|---|
| HX711 | DT | GPIO 16 |
| HX711 | SCK | GPIO 4 |
| Tilt Sensor | OUT | GPIO 17 |
| Temp Sensor | DATA | GPIO 5 |

---

## 📂 Project Structure

```
AI-POWERED-SMARTHYDRATION-TRACKER-USING-IOT-AND-TINY-ML/
│
├── Smart-Hydration-Tracker/     # Source code, firmware, and project files
├── images/                      # Screenshots and diagrams (add here)
└── README.md                    # Project documentation
```

---

## 🚀 Getting Started

### 1. Clone the repository
```bash
git clone https://github.com/DharaniSiva78/AI-POWERED-SMARTHYDRATION-TRACKER-USING-IOT-AND-TINY-ML.git
cd AI-POWERED-SMARTHYDRATION-TRACKER-USING-IOT-AND-TINY-ML/Smart-Hydration-Tracker
```

### 2. Set up the Arduino IDE
- Install the **ESP32 board package**
- Install required libraries: `HX711`, `WiFi`, `PubSubClient` (MQTT), and a TensorFlow Lite Micro library for ESP32

### 3. Configure credentials
Update Wi-Fi and cloud settings in the firmware:
```cpp
const char* ssid     = "YOUR_WIFI_SSID";
const char* password = "YOUR_WIFI_PASSWORD";
// Add your Firebase / AWS IoT configuration here
```

### 4. Calibrate the load cell
Place a known weight on the sensor and adjust the calibration factor until the reading is accurate.

### 5. Upload and run
Select your ESP32 board and port, upload the sketch, then open the Serial Monitor at `115200` baud.

---

## 🖼️ Screenshots

<img width="1050" height="500" alt="image" src="https://github.com/user-attachments/assets/3755d5a3-7aeb-4b47-b8d4-4e5d662dd238" />


<img width="1050" height="500" alt="image" src="https://github.com/user-attachments/assets/38622222-235b-4e35-82bf-254451cb1565" />

<img width="1050" height="500" alt="image" src="https://github.com/user-attachments/assets/288fe48a-a18a-4187-9515-6888b63f4eab" />

<img width="800" height="550" alt="image" src="https://github.com/user-attachments/assets/17306415-5c48-4b63-9cb6-fd38d29c2f4e" />

<img width="1050" height="500" alt="image" src="https://github.com/user-attachments/assets/47a92add-cd12-42b5-a15c-663a84efb105" />

<img width="1050" height="500" alt="image" src="https://github.com/user-attachments/assets/83d0eb97-dfda-4c27-a332-018fbab6f9b6" />

## 🔮 Future Enhancements

- 📱 Mobile app (Android / iOS) with push notifications
- 🧑‍⚕️ Personalized hydration goals based on age, weight, and activity
- 🌦️ Weather-aware hydration recommendations
- ⌚ Wearable and smartwatch integration
- 🗣️ Voice assistant integration
- 🔋 Further power optimization with deep-sleep scheduling
- 📈 Federated learning for privacy-preserving model improvement

---

## 👤 Author

**Dharani Siva**
GitHub: [@DharaniSiva78](https://github.com/DharaniSiva78)

---

<div align="center">

⭐ If you found this project useful, please consider giving it a star!

</div>
