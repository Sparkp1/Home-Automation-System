# Home-Automation-System

A sensor-based home automation system built on an Arduino Uno. It monitors light, temperature, smoke, water level and hand presence, and responds automatically with a fan motor, buzzer alarm, LCD readouts and a touch-free water dispenser.

## Table of Contents

- [Features](#features)
- [How It Works](#how-it-works)
- [Components](#components)
- [Pin Configuration](#pin-configuration)
- [Configurable Parameters](#configurable-parameters)
- [Getting Started](#getting-started)
- [Serial Commands](#serial-commands)
- [Cost Breakdown](#cost-breakdown)
- [Limitations](#limitations)
- [Future Scope](#future-scope)
- [Team](#team)

---

## Features

- **Light-activated system:** an LDR acts as the master switch. The other sensors and actuators only run when the LDR reads below its threshold (i.e. it is covered or in the dark).
- **Automatic cooling:** a DC motor (fan) turns on when the DHT11 reads a temperature above the threshold.
- **Smoke alarm:** an MQ-2 sensor triggers a buzzer when smoke or gas is detected. The alarm can be silenced with a push button or a serial command.
- **Water level monitoring:** an HC-SR04 ultrasonic sensor measures the water level in a tank and shows it on the LCD.
- **Touch-free water dispensing:** an IR obstacle sensor detects a hand near the tap and runs the submersible pump while the hand is present.
- **Rotating LCD display:** a 16x2 I2C LCD cycles through four status pages every 3 seconds.

## How It Works

1. The LDR is read continuously. If the value is **above** the threshold (light detected), the motor, pump and buzzer are switched off and the LCD shows `Light Detected / System OFF`.
2. If the value is **below** the threshold (dark), the system is active and the following run on every loop:

| Function | Input | Condition | Output |
|---|---|---|---|
| Temperature control | DHT11 | Temp > 31 °C | Motor ON |
| Smoke alert | MQ-2 | Analog reading > 200 | Buzzer ON (until reset/muted) |
| Water level | HC-SR04 | Always | Level shown on LCD |
| Water dispensing | IR sensor | Hand detected and tank not empty | Pump ON |

3. After the buzzer is muted, it stays muted until the air clears. Once the smoke reading drops below the threshold, the alarm re-arms.

### LCD Pages

| Page | Line 1 | Line 2 |
|---|---|---|
| 1 | Temperature | Water level (cm) |
| 2 | Tap detection (Yes/No) | Pump status (ON/OFF) |
| 3 | Air quality label | MQ-2 reading + Smoke!/Safe |
| 4 | LDR status (Dark/Light) | LDR value + threshold |

## Components

| Component | Qty | Role |
|---|---|---|
| Arduino Uno | 1 | Main controller |
| Infrared Obstacle Avoidance IR Sensor Module | 1 | Hand detection at the tap |
| Submersible DC 3V Mini Water Pump (vertical) | 1 | Water dispensing |
| 2 ft water pump hose pipe | 1 | Water delivery |
| Photosensitive Resistor (LDR) Module | 1 | System activation |
| DHT11 Temperature & Humidity Sensor Module | 1 | Temperature sensing |
| 3–9 V Short Shaft 180 DC Motor | 1 | Cooling fan |
| TIP120 Power Darlington Transistor | 2 | Switching the motor and pump |
| 1N4007 Diode | 2 | Flyback protection for motor/pump |
| MQ-2 Flammable Gas and Smoke Detector | 1 | Smoke detection |
| Ultrasonic Sensor HC-SR04 | 1 | Water level measurement |
| Active Buzzer Module (5 V) | 1 | Audible alarm |
| 16x2 I2C LCD (address `0x27`) | 1 | Status display |
| Push button | 1 | Buzzer reset |
| Breadboards, jumper wires, external power supply | – | Prototyping and power |

## Pin Configuration

| Arduino Pin | Connected To |
|---|---|
| `D2` | DHT11 data |
| `D3` | HC-SR04 TRIG |
| `D4` | HC-SR04 ECHO |
| `D5` | IR sensor output |
| `D7` | Water pump (via TIP120) |
| `D8` | DC motor (via TIP120) |
| `D9` | Reset button (`INPUT_PULLUP`, active LOW) |
| `D11` | Buzzer |
| `A0` | MQ-2 analog output |
| `A1` | LDR analog output |
| `A4 / A5` | LCD I2C (SDA / SCL) |

## Configurable Parameters

These are defined at the top of the sketch and should be tuned to your setup:

| Constant | Default | Description |
|---|---|---|
| `TEMP_THRESHOLD` | `31` | Temperature (°C) above which the motor turns on |
| `SMOKE_THRESHOLD` | `200` | MQ-2 analog value above which smoke is flagged |
| `LDR_THRESHOLD` | `500` | LDR analog value below which the system is considered "dark" (active) |
| `TANK_HEIGHT_CM` | `10` | Distance from the sensor to the tank bottom, used to compute water level |
| `pageInterval` | `3000` | Milliseconds each LCD page is shown |

The sketch prints the initial LDR value over serial at startup to help with calibration.

## Getting Started

### Requirements

- [Arduino IDE](https://www.arduino.cc/en/software)
- Libraries (install via *Library Manager*):
  - `LiquidCrystal_I2C`
  - `DHT sensor library` (Adafruit) and its dependency `Adafruit Unified Sensor`
  - `Wire` (bundled with the Arduino IDE)

### Setup

1. Wire the circuit as shown in the [wiring diagram](docs/wiring_diagram.png).
2. Clone this repository:
   ```bash
   git clone https://github.com/<your-username>/<your-repo>.git
   ```
3. Open the sketch in the Arduino IDE.
4. If your LCD does not initialise, change its I2C address from `0x27` to `0x3F`.
5. Select **Arduino Uno** and the correct port, then upload.
6. Open the Serial Monitor at **9600 baud** to view calibration output and send commands.

> **Note:** The motor and pump should be powered from an external supply through the TIP120 transistors, not directly from the Arduino pins. Keep the 1N4007 flyback diodes across the motor and pump.

## Serial Commands

| Command | Effect |
|---|---|
| `off` | Turns the buzzer off and mutes it until the air clears |


## Limitations

- **No remote monitoring:** there is no wireless method for remote observation or control.
- **Hardware constraints:** low-cost sensors have limited accuracy, sensitivity and longevity.
- **Power dependency:** the system stops working during a power failure.
- **Limited safety coverage:** only smoke detection is included; gas leakage and intrusion detection are not.

## Future Scope

- **IoT integration:** add Wi-Fi or GSM modules for control and monitoring from a phone or web app.
- **Enhanced safety:** add gas leakage, fire and motion detection.
- **Energy efficiency:** add solar panels or battery backup.
- **User interface:** build a mobile app or GUI with real-time data and manual override.
- **Scalability:** extend to smart buildings, greenhouses or industrial setups.


*Developed as a course project for EEE 4604 at the Islamic University of Technology.*
