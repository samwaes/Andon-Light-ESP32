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

If the ESP32 cannot join a known Wi-Fi network, it will eventually expose a protected fallback AP called `Andon-Setup`. The design target is to keep the same local web UI available on that AP so the four outputs can still be controlled and new Wi-Fi credentials can be entered.

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
│   ├── update-model.md
│   ├── status.md
│   └── bring-up.md
└── esphome/
    ├── andon-light.yaml.example
    └── secrets.yaml.example
```

## Current status

As of 2026-09-19:

- first serial flash completed successfully
- ESPHome 2026.8.2 running on ESP32 rev 3.1
- device online on Wi-Fi
- encrypted ESPHome native API confirmed
- OTA update confirmed by changing the friendly name and reflashing wirelessly
- ESPHome Web OTA component present
- UART programming header soldered
- 12 V Andon not yet connected
- MOSFET GPIO mapping not yet physically verified
- MQTT/UNS not yet configured
- current firmware still has the standard captive-portal behavior from initial bring-up
- next firmware revision will move to fallback AP + local web server + runtime Wi-Fi configuration so direct Andon control remains available without Home Assistant

Next technical steps:

1. Deploy and test the local fallback web interface.
2. Verify `Andon-Setup` and runtime Wi-Fi configuration.
3. Verify browser-based web OTA.
4. Verify GPIO16/17/26/27 against OUT1/OUT2/OUT3/OUT4 with a multimeter.
5. Power the board from 12 V DC and verify that the same supply powers the ESP32.
6. Connect the Andon one channel at a time.
7. Add semantic Andon modes.
8. Add MQTT and the UNS topic model.
9. Validate standalone operation with Home Assistant disconnected.

See [ROADMAP.md](ROADMAP.md) for the implementation plan.
