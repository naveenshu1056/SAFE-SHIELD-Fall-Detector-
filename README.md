# SAFE SHIELD — Elderly Fall Detection and Monitoring Prototype

SAFE SHIELD is an ESP32-based prototype that monitors movement and selected health-related sensor readings. When a high-impact event is detected from the MPU6050 accelerometer, the prototype sets an impact flag, switches a relay output, sends an SMS through a GSM modem, and uploads readings to an IoT endpoint when Wi-Fi is connected.

> **Prototype / safety notice:** This is an educational prototype, not a certified medical device or a validated fall-prevention system. Do not connect it to a real airbag, pyrotechnic inflator, or other hazardous actuator. Test the relay output with an LED or low-voltage dummy load only. The current impact threshold and sensor logic require calibration and validation before any real-world use.

## Features in the supplied prototype

- Fall/impact trigger based on the magnitude of MPU6050 acceleration.
- Relay output activated while the impact flag is set.
- GSM SMS alert using AT commands.
- Wi-Fi HTTP POST upload of sensor values.
- Pulse/SpO₂ readings through the **MAX30100** library used by the supplied code.
- Temperature reading using a DS18B20-compatible OneWire sensor.
- FSR analog input used by the prototype to assign placeholder systolic/diastolic values.
- Battery analog reading and red/green status LEDs.
- FreeRTOS tasks for acceleration monitoring, pulse-oximeter updates, and main logic.

## Hardware listed or implied by the code

- ESP32 development board
- MPU6050 accelerometer/gyroscope (I²C)
- MAX30100 pulse-oximeter module (I²C; this code uses MAX30100, not MAX30102)
- DS18B20 temperature sensor (OneWire)
- FSR connected to an ADC pin
- GSM modem compatible with SIM800-series AT commands
- Relay module (use a safe low-voltage test load)
- Red and green LEDs with suitable current-limiting resistors
- Battery voltage-sense circuit connected to ADC pin 32
- Wi-Fi access point and an IoT server endpoint

## Pin configuration in the supplied code

| Function | ESP32 pin / setting |
|---|---|
| MPU6050 SDA | GPIO 21 |
| MPU6050 SCL | GPIO 22 |
| MAX30100 I²C | `Wire.begin(25, 26, 100000)` — SDA 25, SCL 26 |
| DS18B20 OneWire | GPIO 4 |
| GSM UART RX | GPIO 16 |
| GSM UART TX | GPIO 17 |
| Relay output | GPIO 27 |
| FSR analog input | GPIO 35 |
| Battery analog input | GPIO 32 |
| Red LED | GPIO 14 |
| Green LED | GPIO 5 |

**Important:** The supplied code uses two separate I²C buses: MPU6050 on GPIO 21/22 and MAX30100 on GPIO 25/26. Verify that your wiring matches this arrangement. Add the required pull-up resistors and power-level compatibility according to each breakout board's documentation.

## Software requirements

- Arduino IDE 2.x (or a compatible Arduino build environment)
- ESP32 board support package
- Libraries:
  - `MAX30100_PulseOximeter`
  - `OneWire`
  - `DallasTemperature`
  - ESP32 core libraries: `Wire`, `WiFi`, `HTTPClient`, `WiFiClientSecure`, `HardwareSerial`
- A GSM modem and active SIM card for SMS testing
- A configured IoT endpoint that accepts URL-encoded POST data

Install third-party libraries through **Arduino IDE → Library Manager** where available. Library names can vary by publisher; select the library that provides the exact include file used by the sketch.

## Setup steps

1. Install Arduino IDE and the ESP32 board package.
2. Install the MAX30100, OneWire, and DallasTemperature libraries.
3. Open your prototype `.ino` sketch.
4. Check all wiring against the pin table above.
5. Replace the Wi-Fi, phone-number, and IoT endpoint placeholders in the sketch. Do not publish real credentials in a public repository.
6. Configure the server to accept the fields listed below.
7. Select the correct ESP32 board and COM/serial port.
8. Upload the sketch and open Serial Monitor at **115200 baud**.
9. Test each sensor individually, then test SMS and network upload.
10. Test the relay using an LED or safe low-voltage dummy load only.

## Configuration / environment variables

Arduino sketches do not automatically load `.env` files. For local development, keep settings in a separate untracked `secrets.h` file or replace placeholders before compiling. For example:

```cpp
// secrets.h — keep this file out of version control
#pragma once
const char* WIFI_SSID = "YOUR_WIFI_SSID";
const char* WIFI_PASSWORD = "YOUR_WIFI_PASSWORD";
const char* IOT_POST_URL = "https://your-server.example/post_value.php";
const char* ALERT_PHONE_NUMBER = "+CCXXXXXXXXXX";
```

Then include `secrets.h` and update the sketch to use these constants. The supplied prototype currently hardcodes Wi-Fi credentials, an endpoint, and a recipient number; move them into configuration before sharing the code.

Suggested configuration values:

| Setting | Purpose |
|---|---|
| `WIFI_SSID` | Wi-Fi network name |
| `WIFI_PASSWORD` | Wi-Fi password |
| `IOT_POST_URL` | HTTPS endpoint receiving sensor data |
| `ALERT_PHONE_NUMBER` | Recipient of the GSM SMS alert |
| `IMPACT_THRESHOLD_G` | Initial prototype threshold; currently `4.0` in code |
| `IMPACT_HOLD_MS` | Impact flag hold time; currently `13000` ms |

These are documentation suggestions; the current sketch does not yet read all of these from environment variables or a configuration file.

## IoT upload format

The prototype sends `application/x-www-form-urlencoded` data every 2 seconds while Wi-Fi is connected:

| Field | Meaning in prototype |
|---|---|
| `value1` | Impact flag (`0` or `1`) |
| `value2` | Temperature in °C |
| `value3` | Heart rate value |
| `value4` | SpO₂ value |
| `value5` | Prototype systolic value |
| `value6` | Prototype diastolic value |

The FSR logic currently assigns `115/75` when its ADC value is at least 3800 and `0/0` otherwise. **These values are placeholders, not measured blood pressure.** Do not represent them as clinically valid blood-pressure readings.

## Known limitations to address

- Acceleration magnitude alone can miss falls and can also trigger on non-fall impacts. Validate thresholds with safe, controlled tests.
- The code uses `WiFiClientSecure::setInsecure()`, which disables certificate verification. Use proper certificate validation before deployment.
- The sketch blocks while waiting for Wi-Fi during startup and has limited connection/error handling.
- SMS success is not verified by reading modem responses.
- The MAX30100 readings need signal-quality checks and sensor-contact validation.
- The FSR-to-blood-pressure mapping is not a valid blood-pressure measurement.
- Battery ADC thresholds depend on the actual voltage-divider and ESP32 ADC calibration.
- An actual wearable airbag requires a separately engineered, safety-reviewed inflation mechanism. This code only switches a relay output.

## Repository contents

- `README.md` — setup and configuration
- `docs/TECHNICAL_DESIGN.md` — architecture, data flow, and design notes
- `tests/SAMPLE_DATA.csv` — illustrative sample records
- `tests/TEST_CASES.md` — manual prototype test plan
- `LICENSE` — MIT License

## Suggested next development steps

1. Separate secrets from the sketch and remove them from source control.
2. Add sensor initialization checks and reconnect logic.
3. Log raw acceleration and validate detection thresholds using labelled test data.
4. Add a cancel/acknowledge button if available in the hardware.
5. Secure the HTTPS endpoint and verify server responses.
6. Treat health readings as indicative only unless validated against appropriate reference devices.
