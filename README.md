# DAQ, Power Supply & Sensor Feedthrough for a Low-Vacuum Test Chamber

Project work for the Chair of Spacecraft Systems, Technical University of Munich (TUM).

## Overview

This project supports a low-vacuum chamber that uses only a primary pump, built for small-scale, quick tests of different payloads. An NVIDIA Jetson was used as a representative test payload. My contribution focused on two tasks:

1. **Temperature measurement:** Identify a low-cost, suitable method for reading payload component temperatures with a 4× PT1000 sensor setup.
2. **Power supply:** Provide a continuous power supply method for the payload.

I evaluated several alternative approaches for each task. The only components purchased were PT1000 RTD sensors, PTFE-insulated wires, solder-type D-sub connectors, an Arduino UNO R4 WiFi, and a Raspberry Pi.

> **Note:** This repository gives an overview of the approach rather than a step-by-step build guide. Some details are withheld due to project restrictions.

## System Architecture

```
 VACUUM CHAMBER                                    AMBIENT SIDE
+----------------------------------+
|  Payload (e.g. Jetson)           |
|  [PT1000 x4 on components]       |
|        |                         |
|  [D-sub, vacuum side] -----------+--> [D-sub, ambient side] -> [MAX31865 boards] -> [Arduino UNO R4 WiFi]
|                                  |                                                          |
|  Jetson internal sensors         |                                                          | serial
+----------|-----------------------+                                                          v
           |  local Wi-Fi hotspot                                                      [Raspberry Pi]
           +------------------------------------------------------------------------>         |
                                                                                                v
                                                                         [Grafana dashboard] <- [InfluxDB]
```

- **Payload:** Any small payload under test. An NVIDIA Jetson was used as the example payload.
- **External temperature sensing:** Four PT1000 sensors are mounted on the payload's components and soldered to a D-sub connector on the vacuum side.
- **Feedthrough harness:** The mating D-sub on the ambient side connects to MAX31865 RTD-to-digital converter boards.
- **Acquisition:** An Arduino UNO R4 WiFi reads the MAX31865 boards over SPI and sends the readings to a Raspberry Pi via serial.
- **Payload telemetry:** The Raspberry Pi communicates with the Jetson over a local Wi-Fi hotspot and retrieves the temperatures from the Jetson's own internal sensors.
- **Data pipeline:** A Python bridge script on the Raspberry Pi writes all data (PT1000 and Jetson internal readings) to an InfluxDB database.
- **Visualization:** Grafana reads from InfluxDB and displays everything on one dashboard.
- **Remote Connection:** Tailscale is used to remote connection to the raspberry pi and if needed to jetson while tvac test is ongoing. Also for restricting everyone to have access to test data.

## Key Features

- **Low-cost sensing:** Off-the-shelf PT1000 sensors with MAX31865 boards instead of a commercial DAQ system.
- **Two data sources, one dashboard:** External PT1000 readings and the payload's internal sensor readings are shown side by side, which allows cross-checking.
- **Plug-and-play sensors:** The Arduino firmware re-scans all 6 MAX31865 channels every cycle, so sensors can be connected or removed without re-uploading code. Only detected sensors appear in the output.
- **Vacuum-side / ambient-side split:** The D-sub harness separates the vacuum-side wiring from the electronics, so payloads can be swapped easily.
- **Live monitoring:** Continuous logging to InfluxDB with a Grafana dashboard.

## Tech Stack

| Area | Tools |
|---|---|
| Test payload | NVIDIA Jetson |
| External sensors | PT1000 RTDs, PTFE wiring |
| Signal conditioning | MAX31865 (2-wire configuration, 4.3 kΩ reference resistor) |
| Microcontroller | Arduino UNO R4 WiFi |
| Edge computing | Raspberry Pi |
| Wireless link | Local Wi-Fi hotspot (Pi to Jetson) |
| Database | InfluxDB |
| Visualization | Grafana |
| Connectors | D-sub (solder type) |

## Hardware Details

**1. PT1000 connected to the MAX31865 (2-wire jumper configuration)**

![MAX31865 2-wire jumper setup](images/max31865_2wire_jumper.jpeg)

**2. D-sub mating connector with labeled sensors**

![Pin mapping of the PT1000 to the mating D-sub connector](images/Pin_Labelling_PT1000_harness.jpeg)

**3. Arduino wiring**

Shown for two sensors; the same scheme was applied to all four.

![Arduino to PT1000 wiring diagram](images/arduino_to_pt1000_wiring.png)

## Dashboard

Grafana dashboard showing the PT1000 readings and the payload's internal temperatures:

![Grafana dashboard overview](images/Grafana_Img1.png)

![Grafana dashboard overview](images/Grafana_Img2.png)

## Results

- **Vacuum level:** Payloads were tested at pressures down to 0.8 Pa (8×10⁻³ mbar), the ultimate pressure of the primary pump.
- **Sampling rate:** About 1 Hz per PT1000 channel, with up to 6 channels supported (4 used in the vacuum test), logged continuously to InfluxDB.
- **Cost:** Built from low-cost, off-the-shelf components.

