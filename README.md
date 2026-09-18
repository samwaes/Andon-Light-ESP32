# Andon Light ESP32

Portable ESP32-based Andon light and MQTT/UNS demonstrator.

## Project goal

Build one physical Andon device that can be used in several ways without reflashing the ESP32 for every environment:

- Home Assistant via the native ESPHome API
- Standard MQTT over Wi-Fi, without Home Assistant
- Industrial lab and UNS demonstrations through an MQTT broker
- Local device logic so the light still behaves predictably if Home Assistant is unavailable
- Optional local web interface for commissioning and fallback control

The design principle is to keep the physical device, Home Assistant integration, and industrial MQTT namespace separate.

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
- USB-C power input
- ESP32-WROOM-32E module
- exposed GPIO headers
- IO0 boot button
- separate UART programming header
- programming header labels: 5V, TX, RX, GND, GND, IO0

The exact GPIO-to-MOSFET mapping is not yet confirmed and must be measured or tested before the final ESPHome configuration is locked.

### USB-to-UART adapter

Received adapter exposes:

- 3V3
- GND
- +5V
- TXD
- RXD
- DTR

Initial programming connection:

```text
USB-UART            ESP32 board
TXD       --------> RX
RXD       --------> TX
GND       --------> GND
```

The ESP32 board should be powered separately during initial flashing. Do not connect the adapter's 5 V or 3.3 V power pins when the ESP32 board is already powered.

To enter bootloader mode, hold IO0 low during power-up, either with the IO0 button or by temporarily connecting IO0 to GND.

## Target architecture

```text
                         ESP32 / ESPHome
                              |
                            Wi-Fi
                              |
          +-------------------+-------------------+
          |                   |                   |
     ESPHome API            MQTT            Local web UI
          |                   |
          v                   v
   Home Assistant       MQTT broker
                            |
                 +----------+----------+
                 |          |          |
              Node-RED   UNS demo   MQTT Explorer
```

ESPHome is the firmware platform. Home Assistant is one client. MQTT is the portable industrial interface.

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
│   └── bring-up.md
└── esphome/
    ├── andon-light.yaml.example
    └── secrets.yaml.example
```

## Current status

Project phase: hardware received, architecture defined, first flash not yet performed.

Next technical steps:

1. Solder the six-pin UART programming header.
2. Verify the USB-UART adapter is operating at 3.3 V logic.
3. Perform the first ESPHome flash.
4. Determine the GPIO mapping for OUT1 to OUT4.
5. Verify the MOSFET output polarity and terminal behavior with a multimeter.
6. Connect the 12 V Andon.
7. Add Home Assistant native API control.
8. Add MQTT and the UNS topic model.
9. Validate operation with Home Assistant disconnected.

See [ROADMAP.md](ROADMAP.md) for the implementation plan.
