# Andon Light ESP32

Portable ESP32-based Andon light and MQTT/UNS demonstrator.

## Project goal

Build one physical Andon device that can be used in several ways without reflashing the ESP32 for every environment:

- Home Assistant via the native ESPHome API
- Standard MQTT over Wi-Fi, without Home Assistant
- Industrial lab and UNS demonstrations through an MQTT broker
- Local device logic so the light still behaves predictably if Home Assistant is unavailable
- Local web interface for direct Andon control and commissioning
- Fallback Wi-Fi access point for use when no known network is available
- Runtime Wi-Fi provisioning from the local web interface
- Multiple firmware update paths: UART recovery, ESPHome OTA and browser-based web OTA

The design principle is to keep the physical device, Home Assistant integration, local fallback control and industrial MQTT namespace separate.

## Current hardware

### Andon light

HNTD TD-50, 12 V DC, three-color LED warning light with buzzer.

Observed wiring from the product labels:

| Wire | Function |
| --- | --- |
| Brown | Common +12 V |
| Red | Red light, switched to 0 V |
| Yellow | Yellow light, switched to 0 V |
| Green | Green light, switched to 0 V |
| Orange | Buzzer, switched to 0 V |

The lamp is the 12 V constant-light version.

### Controller

ESP32-WROOM-32E quad MOSFET switch board.

Observed on the received board:

- 4 MOSFET channels, OUT1 to OUT4
- N-channel MOSFET low-side switching architecture
- 5-60 V DC input
- on-board conversion to power the ESP32 from the DC input
- USB-C 5 V power input
- ESP32-WROOM-32E module
- exposed GPIO headers
- IO0 boot button
- separate UART programming header
- programming header labels: 5V, TX, RX, GND, GND, IO0
- six-pin UART header now soldered

The expected MOSFET mapping for this board family is currently:

| Output | Expected GPIO | Planned Andon function |
| --- | ---: | --- |
| OUT1 | GPIO16 | Red |
| OUT2 | GPIO17 | Yellow |
| OUT3 | GPIO26 | Green |
| OUT4 | GPIO27 | Buzzer |

This mapping still needs physical verification before the Andon is connected.

### USB-to-UART adapter

Silicon Labs CP210x USB-to-UART adapter, detected by Windows as COM7 during initial bring-up.

Initial programming connection:

```text
USB-UART            ESP32 board
TXD       --------> RX
RXD       --------> TX
GND       --------> GND
```

The ESP32 board is powered separately during serial flashing. Do not connect the adapter's 5 V or 3.3 V power pins when the ESP32 board is already powered.

## Target architecture

```text
                         ESP32 / ESPHome
                              |
                            Wi-Fi
                              |
          +-------------------+-------------------+
          |                   |                   |
     ESPHome API            MQTT            Local web UI
          |                   |                   |
          v                   v                   v
   Home Assistant       MQTT broker       Direct control
                            |
                 +----------+----------+
                 |          |          |
              Node-RED   UNS demo   MQTT Explorer
```

If the ESP32 cannot join a known Wi-Fi network, it exposes a protected fallback AP called `Andon-Setup`. The same local web UI is used for Andon control, Wi-Fi provisioning and, from firmware 0.5.0 onward, runtime MQTT broker configuration. This allows the device to be moved to an industrial environment and pointed at that site's MQTT broker without reflashing.

## MQTT / UNS concept

Initial namespace:

```text
hupla/demo/factory01/line01/andon01/
```

Planned logical topics include:

```text
.../command/mode
.../command/outputs
.../state/mode
.../state/outputs
.../status/online
.../event
```

The device should expose semantic states such as:

- OFF
- RUNNING
- WARNING
- FAULT
- STOPPED
- MAINTENANCE

instead of requiring external systems to know GPIO numbers.

## Repository structure

```text
.
├── README.md
├── ROADMAP.md
├── docs/
│   ├── architecture.md
│   ├── hardware.md
│   ├── firmware.md
│   ├── mqtt-uns.md
│   ├── alarm-philosophy.md
│   ├── update-model.md
│   ├── status.md
│   └── bring-up.md
└── esphome/
    ├── andon-light.yaml.example
    └── secrets.yaml.example
```

## Current status

As of 2026-09-19:

- ESP32-WROOM-32E controller and 12 V HNTD TD-50 Andon are operational
- GPIO16 / GPIO17 / GPIO26 / GPIO27 drive red / yellow / green / buzzer through the four MOSFET outputs
- local ESPHome web portal, fallback architecture, encrypted native API and OTA are operational
- semantic Andon modes, flashing patterns, acknowledge behavior and Buzzer Mute Override are deployed and working
- the TD-50 red/yellow interaction is confirmed as an internal Andon limitation, so semantic modes use one color at a time
- firmware 0.5.0 is deployed
- MQTT broker address, port, username, password and topic prefix can be configured from the Andon web interface without reflashing
- the device connects successfully to the Home Assistant Mosquitto broker at 192.168.129.15:1883
- MQTT subscribe/listen and publish control have been verified from Home Assistant
- standard ESPHome MQTT topics can currently read and change Andon mode, mute and acknowledge behavior
- custom semantic UNS topics are deliberately deferred for now

Current baseline is therefore a working portable Andon controller with local control, Home Assistant control and standard MQTT control.

Next technical steps, when the project is resumed:

1. Verify MQTT settings survive a full power cycle.
2. Point the Andon at a second MQTT broker from the web interface to prove site portability.
3. Validate standalone operation with Home Assistant unavailable.
4. Verify browser-based web OTA.
5. Later, decide whether to add custom semantic MQTT/UNS topics, retained state and birth/last-will conventions.

See [ROADMAP.md](ROADMAP.md) for the implementation plan.
