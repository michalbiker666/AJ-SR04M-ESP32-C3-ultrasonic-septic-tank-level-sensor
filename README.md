# AJ-SR04M ESP32-C3 Septic Tank Level Sensor (ESPHome)

An ultra-low-power, DIY smart home solution for monitoring septic tank levels. This project is built using **ESPHome** and integrates seamlessly with **Home Assistant**.

## 🚀 Key Features

* **Deep Sleep Optimization:** The device wakes up once every 24 hours, performs a measurement, and goes back to sleep to maximize battery life.
* **Sequential Logic:** To handle voltage drops on small batteries (e.g., 18650 with TP4056), the device performs the ultrasonic measurement *before* enabling the power-hungry WiFi radio.
* **Advanced Data Filtering:** Uses a median-of-three filter and a "warm-up" pulse to ensure high accuracy and eliminate random noise from the ultrasonic sensor.
* **Service Window:** Features a 10-second delay on boot, allowing for easy OTA (Over-The-Air) or USB updates before the device enters deep sleep.
* **Failsafe:** Safe Mode is disabled to prevent the status LED from draining the battery during minor voltage fluctuations.

## 🛠 Hardware

* **Microcontroller:** ESP32-C3 Super Mini.
* **Sensor:** AJ-SR04M (Waterproof Ultrasonic Sensor).
* **Power:** 18650 Li-ion battery + TP4056 charging module.
* **Stabilization:** 470µF capacitor across VCC/GND to handle WiFi power spikes.
* **Modification:** Status LEDs physically removed from the board to achieve true micro-ampere consumption during sleep.

## 🔌 Wiring

| ESP32-C3 Pin | Sensor Pin | Function |
| :--- | :--- | :--- |
| **5V / VBUS** | VCC | Power (from TP4056/Battery) |
| **GND** | GND | Ground |
| **GPIO3** | RX | UART TX (Trigger) |
| **GPIO4** | TX | UART RX (Echo) |

## ⚙️ How it Works

1. **Boot:** Wakes up and waits for 10 seconds (Service Window).
2. **WiFi Sync:** Connects to the local network to prepare for data transmission.
3. **Measurement:**
    * Triggers the AJ-SR04M via UART (0x55).
    * Discards the first reading (warm-up pulse).
    * Collects 3 valid readings.
    * Calculates the **median** to ignore outliers and interference.
4. **Publish:** Sends the distance (cm), level (%), and status (OK/High/Full) to Home Assistant.
5. **Sleep:** Enters Deep Sleep for 24 hours.

## 📝 Lessons Learned (Pro-Tips)

* **The LEDS Trap:** On many ESP32-C3 Super Mini boards, the red and the blue LED (GPIO8) may glow during Deep Sleep due to the pin's floating state. Physical removal is the most reliable way to save energy.
* **WiFi vs. Battery:** Always use `fast_connect: true` in ESPHome to minimize the time the radio is active.
* **UART Stability:** The AJ-SR04M requires a specific 0x55 trigger in Mode 4. The included Lambda handles the checksum verification for every packet.
