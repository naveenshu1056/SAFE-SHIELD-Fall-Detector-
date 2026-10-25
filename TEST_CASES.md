# SAFE SHIELD — Sample Test Cases

These are manual, safe prototype tests based on the supplied sketch. Sample data is illustrative, not collected clinical data. Use a relay LED or low-voltage dummy load only.

| ID | Test | Procedure | Expected result | Pass criteria |
|---|---|---|---|---|
| TC-01 | Startup and serial | Power ESP32 and open Serial Monitor at 115200 baud | Sketch starts; serial output appears | No unexpected reboot; output is readable |
| TC-02 | MPU6050 connection | Verify I²C wiring and observe acceleration values during controlled movement | Values change with orientation/movement | Readings are plausible and stable |
| TC-03 | No-impact condition | Keep sensor stationary and avoid sudden movement | Impact flag remains 0; relay output LOW | No relay activation during the observation period |
| TC-04 | Impact threshold | In a controlled bench setup, safely create a brief sensor movement/impact without dropping hardware | If magnitude exceeds the code threshold, flag becomes 1 and relay goes HIGH | Trigger and reset are observable; record false triggers |
| TC-05 | Impact timeout | Trigger the flag and then keep the sensor still | Flag clears after about 13 seconds from the last qualifying event; relay returns LOW | Output resets as implemented |
| TC-06 | GSM SMS | Use a valid SIM, antenna, coverage, and configured recipient; monitor modem | SMS alert text is sent when an impact event occurs | Test phone receives the alert; document modem/network failures |
| TC-07 | Wi-Fi upload | Connect to configured Wi-Fi and a test endpoint | POST is attempted about every 2 seconds | Server receives `value1` through `value6` |
| TC-08 | Wi-Fi unavailable | Test with access point unavailable | Upload is skipped when Wi-Fi is disconnected | Device remains responsive; reconnect behavior is documented |
| TC-09 | MAX30100 readings | Use correct sensor wiring and place finger on sensor | Library updates; valid-range readings may be reported | Values are treated as estimates and compared with a reference only for demonstration |
| TC-10 | Temperature sensor | Connect DS18B20 and compare with a room thermometer | Temperature is read from the first OneWire device | Reading is plausible; missing sensor behavior is recorded |
| TC-11 | FSR threshold | Apply varying pressure to FSR while watching ADC-derived result | At ADC >= 3800, code sets placeholder values 115/75; otherwise 0/0 | Logic behaves as coded; do not call this measured blood pressure |
| TC-12 | Battery indicator | Apply safe known ADC inputs / verify divider output | ADC average below 2700: red LED; otherwise green LED | LED matches threshold logic; confirm thresholds for actual circuit |

## Test record template

For each test, record:

- Date / tester:
- Hardware revision and wiring:
- Firmware revision:
- Test case ID:
- Observed output:
- Pass / fail:
- Notes, errors, and corrective action:

## Important interpretation notes

- The code detects an acceleration magnitude above 4.0 g; this alone does not prove a person has fallen.
- Test results should not be described as clinical validation.
- The FSR-derived 115/75 values are hard-coded placeholders, not actual blood pressure.
- Never test by intentionally falling or by connecting this prototype to a real airbag inflator.
