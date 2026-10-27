Heatwave Guardian 🌡️
An Arduino UNO Q-based environmental monitoring and automatic cooling prototype that reads indoor and outdoor conditions, displays sensor data, controls a low-voltage DC fan, and provides an audible warning when outdoor temperature exceeds a configured threshold.
Project status: Prototype. The current implementation uses sensor readings and programmed rules. A trained AI/ML model is not included unless you add, train, and test one separately.

Features
- Indoor monitoring: SHT41 temperature and relative humidity readings.
- Outdoor temperature: Waterproof DS18B20 probe.
- Additional readings: DHT11 temperature and humidity.
- Local display: SSD1306 128×64 I²C OLED.
- Fan control: L298N motor driver controls a compatible low-voltage DC fan.
- Over-temperature alert: Active buzzer turns on when a valid DS18B20 reading is greater than 50 °C.
- Dashboard communication: Python application exchanges sensor values and fan commands with the Arduino sketch using Arduino RouterBridge.
- Potential extension: Solar-assisted battery operation and a trained sensor-classification model can be explored after implementation and testing.
System Overview
 SHT41 ─────────┐
 DS18B20 ──────┼──> Arduino UNO Q ─────> OLED display
 DHT11 ────────┘          │
                          ├────────────> Buzzer (outdoor temp > 50 °C)
                          │
                          ├────────────> L298N ─────> Low-voltage DC fan
                          │
                          └<── Arduino RouterBridge ──> Python dashboard
How it works
1. The Arduino sketch reads the sensors at approximately two-second intervals.
2. Valid readings are displayed on the OLED and exposed through RouterBridge.
3. The Python application can show readings, calculate/display heat-risk information, and send fan ON/OFF commands.
4. The L298N switches the DC fan according to the command received by the Arduino.
5. The Arduino activates the buzzer when the valid outdoor DS18B20 reading is above 50 °C; at 50 °C or below, it switches the buzzer off.
6. The cycle repeats continuously.
Note: Automatic fan operation depends on the fan-control logic in the Python application calling the bridge function. The Arduino sketch does not automatically turn the fan on merely because a risk label says HIGH, unless that behaviour is explicitly implemented.
Hardware
Component	Purpose
Arduino UNO Q (2 GB version, if used)	Main controller and application platform
SHT41 module	Indoor temperature and relative humidity
DS18B20 waterproof probe	Outdoor temperature
DHT11 module	Additional temperature and humidity readings
SSD1306 128×64 I²C OLED	Local display
L298N motor driver	Low-voltage DC fan control
Active buzzer module	Audible temperature alert
DC fan(s)	Air movement for the prototype
4.7 kΩ resistor	DS18B20 data-line pull-up
Jumper wires and breadboard	Prototyping connections
Battery pack and solar panel (if installed)	Potential portable/solar-assisted power


Pin Connections
Confirm the exact pin labels and electrical requirements for your UNO Q board and each module before wiring.
Signal / module	UNO Q connection
DS18B20 data	D2
DHT11 data	D3
Active buzzer signal	D4
L298N IN1	D5
L298N IN2	D6
SHT41 SDA/SCL	Board I²C pins
OLED SDA/SCL	Board I²C pins, shared bus where supported
Sensor/display ground	GND, as required
L298N ground	Common ground with controller and motor supply, as required


Wiring notes
- A DS18B20 normally needs a 4.7 kΩ pull-up resistor between DATA and a suitable logic supply.
- Check that each sensor/display breakout is compatible with the UNO Q's logic voltage.
- Power the fan from a suitable external DC supply through the L298N; do not power a high-current fan from a GPIO pin.
- Check the L298N's current rating, voltage drop, cooling, and the fan's startup current.
- The L298N is for compatible DC loads only. Never connect a 220–240 V AC fan to the L298N.
- A solar panel and battery require an appropriate regulator/charger and protection. Do not connect them directly without checking voltage, polarity, battery chemistry, and charging requirements.
Software
- Arduino IDE and the required board support package
- Arduino sketch using Arduino_RouterBridge
- Python application (main.py) for dashboard and application logic
- Libraries used by the sketch:
  - Arduino_RouterBridge
  - Adafruit_SHT4x
  - OneWire
  - DallasTemperature
  - DHT
  - U8g2
Install the libraries through the Arduino IDE Library Manager where available, and use versions compatible with your installed UNO Q board package.
Repository Structure
A suggested GitHub repository layout:
heatwave-guardian/
├── README.md
├── arduino/
│   └── HeatwaveGuardian.ino
├── app/
│   └── main.py
├── docs/
│   ├── block-diagram.png
│   ├── circuit-diagram.png
│   └── project-report.pdf
├── images/
│   ├── prototype.jpg
│   └── dashboard.png
└── LICENSE
Adjust file names and folders to match the files actually included in your repository.
Setup
1. Wire the hardware
Connect the sensors, OLED, buzzer, and L298N using the pin table above. Check power requirements and common-ground arrangements before applying power.
2. Upload the Arduino sketch
1. Open arduino/HeatwaveGuardian.ino in Arduino IDE (or open the actual sketch file you use).
2. Select the correct Arduino UNO Q board and port.
3. Install the required libraries.
4. Upload the sketch.
5. Open Serial Monitor at the baud rate used by your sketch (the supplied sketch uses 115200).
3. Start the Python application
1. Open the Python application folder in your UNO Q development environment.
2. Check that the RouterBridge function names used by Python match those registered by the Arduino sketch.
3. Start the application using the method supported by your installed App Lab / UNO Q environment.
4. Confirm that readings arrive and that fan commands change the fan state.
The exact Python launch command and dependency installation steps depend on the contents of your main.py and the environment you use. Add those commands here once verified.
RouterBridge Functions
The Arduino sketch exposes these functions for the Python application:
Function	Purpose
get_sht_temp	Read SHT41 temperature
get_sht_rh	Read SHT41 relative humidity
get_probe_temp	Read DS18B20 temperature
get_dht_temp	Read DHT11 temperature
get_dht_rh	Read DHT11 relative humidity
get_fan	Read current fan state
set_fan	Request fan ON/OFF
set_risk	Update the risk label shown on the OLED
set_forecast	Update the forecast value shown on the OLED


Make sure the Python app uses the same names and data types.
Control Logic
Fan
The fan state is controlled by the application through set_fan(True) or set_fan(False). The Python code should apply the project's intended indoor-risk policy and send the appropriate command.
Buzzer
- DS18B20 reading > 50 °C: buzzer ON.
- DS18B20 reading ≤ 50 °C: buzzer OFF.
- Invalid/disconnected probe: buzzer should remain OFF and the sensor fault should be reported.
The 50 °C value is a configured project threshold, not a universal health-safety threshold.
AI/ML Status
The current described implementation is rule-based. No trained AI/ML model should be claimed unless it has been trained, integrated, and tested.

Possible future work:
- Train a compact Edge Impulse classifier using labelled temperature/humidity time-series data to classify environmental conditions.
- Consider FOMO or YOLO only if a compatible camera is added for visual person/object detection. These are vision models and do not directly classify heat risk from temperature sensors.
- Record sensor data and validate any model on data not used during training.
Testing Checklist
- [ ] SHT41 readings appear and are plausible.
- [ ] DS18B20 readings appear and invalid/disconnected states are handled.
- [ ] DHT11 readings appear and invalid values are handled.
- [ ] OLED updates correctly.
- [ ] Python dashboard receives sensor readings.
- [ ] set_fan(True) turns the DC fan on.
- [ ] set_fan(False) turns the DC fan off.
- [ ] Buzzer turns on above 50 °C and off at or below 50 °C.
- [ ] The system runs continuously without unexpected resets.
- [ ] Fan supply and L298N temperature/current remain within their ratings.
- [ ] Solar/battery operation is tested separately before it is described as ready.
Limitations
- The current design uses configured rules unless a trained model is separately integrated.
- Sensor accuracy depends on placement, calibration, and environmental conditions.
- The DHT11 is a basic sensor and may be less accurate than the SHT41.
- A fan alone does not guarantee safe conditions during extreme heat.
- Solar/battery runtime and charging safety require verification.
- The prototype is not a certified medical, industrial, or life-safety device.
Future Improvements
- Add a reliable data-logging and trend-visualization feature.
- Train and validate a compact sensor-based ML classifier.
- Add camera-based person detection only if it supports a clearly defined use case.
- Improve enclosure, wiring strain relief, and electrical protection.
- Measure actual power consumption and battery runtime.
- Add sensor-fault alerts and a documented fail-safe policy.
- Test the finished unit over extended operating periods.
Estimated Cost
The latest component estimate discussed for the prototype was approximately ₹8,943–₹9,063 before enclosure, assembly, calibration, packaging, taxes, shipping, and other overheads. This estimate depends on whether the stated fan price is per fan or for both fans. Replace estimates with actual bills/quotes before publishing a final bill of materials.
Safety
- Disconnect power before changing wiring.
- Do not connect mains AC to the L298N.
- Use a correctly rated DC supply for the fan.
- Verify battery charging and solar regulation before use.
- Do not rely on this prototype as the sole protection against heat illness or as a certified safety alarm.
Contributing
Suggestions and improvements are welcome. If you contribute, please include the hardware revision, software/library versions, steps to reproduce issues, and test results.
License
Choose a license before publishing this repository. For example, MIT is commonly used for open-source software, but hardware files, images, and documentation may need separate terms. Add a LICENSE file only after deciding which terms you want to apply.
Project: Heatwave Guardian
Platform: Arduino UNO Q + Python / Arduino RouterBridge
Purpose: Environmental monitoring, rule-based fan control, and outdoor over-temperature alert
