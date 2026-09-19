# Andon Light ESP32

Portable ESP32-based industrial Andon demonstrator with local control, Home Assistant integration and runtime-configurable MQTT.

Last reviewed: 2026-09-19

## Current baseline

Firmware 0.5.0 is deployed and working on the real hardware.

Verified:

- ESP32-WROOM-32E quad-MOSFET controller running from the 12 V supply
- HNTD TD-50 red/yellow/green tower with buzzer connected and operational
- GPIO16 -> OUT1 -> red
- GPIO17 -> OUT2 -> yellow
- GPIO26 -> OUT3 -> green
- GPIO27 -> OUT4 -> buzzer
- local ESPHome Web Server v3 working
- protected fallback AP called `Andon-Setup`
- encrypted ESPHome native API working with Home Assistant
- ESPHome OTA working
- semantic Andon modes, flash patterns, acknowledge and Buzzer Mute Override working
- Mosquitto MQTT connection working
- MQTT broker address, port, username and password configurable from the local web interface without reflashing
- MQTT listen and publish verified from Home Assistant
- standard ESPHome MQTT command and state topics working

Not yet verified:

- broker settings after a full power cycle
- connection to a second MQTT broker
- operation with Home Assistant deliberately unavailable
- browser-based Web OTA
- final fuse, strain relief and enclosure

Custom semantic UNS topics are deliberately deferred. The current standard ESPHome MQTT interface is the working project baseline.

## Project objective

The project demonstrates one physical Andon that can be moved between home, lab and industrial environments without changing its basic firmware for every location.

The device supports four separate interaction paths:

```text
                        ESP32 / ESPHome
                              |
          +-------------------+-------------------+
          |                   |                   |
   local web UI         ESPHome native API       MQTT
          |                   |                   |
 commissioning          Home Assistant        Mosquitto
 direct control                              other clients
```

Home Assistant is useful but is not intended to be mandatory for the industrial MQTT path.

## Hardware

### Andon

HNTD TD-50, 12 V DC, constant-light version.

| Wire | Function |
| --- | --- |
| Brown | common +12 V |
| Red | red lamp control |
| Yellow | yellow lamp control |
| Green | green lamp control |
| Orange | buzzer control |

The unit uses a common positive supply and individual low-side control wires.

### Controller

ESP32-WROOM-32E quad MOSFET board with NCE6020AK MOSFETs.

Confirmed mapping:

| Output | ESP32 GPIO | Andon function |
| --- | ---: | --- |
| OUT1 | GPIO16 | Red |
| OUT2 | GPIO17 | Yellow |
| OUT3 | GPIO26 | Green |
| OUT4 | GPIO27 | Buzzer |

The board and Andon are powered from the same 12 V supply.

See [docs/hardware.md](docs/hardware.md) for the complete hardware record.

## Andon behavior

Normal applications use semantic modes rather than direct GPIO control.

Current modes:

```text
OFF
READY
RUNNING
STARTING
ATTENTION
WARNING
URGENT_WARNING
FAULT
CRITICAL
EMERGENCY
STOPPED
MAINTENANCE
MANUAL
```

The ESP32 translates each mode locally into the required color, flash rate and buzzer pattern.

Acknowledge silences the audible alarm and changes active warning/fault flashing to steady without clearing the mode.

Buzzer Mute Override suppresses the physical buzzer without changing the process mode or acknowledge state.

See [docs/alarm-philosophy.md](docs/alarm-philosophy.md).

## Known hardware limitation

The real TD-50 has a confirmed internal interaction between red and yellow:

- red + green works
- yellow + green works
- red + yellow does not work reliably
- the behavior follows the Andon function when output channels are swapped

Semantic modes therefore use one color at a time. Multi-color control remains available only for diagnostics.

## Runtime commissioning

### Wi-Fi

The firmware contains the home Wi-Fi as its initial known network.

If that network is unavailable, the device starts:

```text
Andon-Setup
```

The local web interface can then be used to enter a new SSID and password through `wifi.configure`.

### MQTT

Firmware 0.5.0 exposes these fields in the local web interface:

- MQTT Broker
- MQTT Port
- MQTT Username
- MQTT Password
- MQTT Topic Prefix
- Save & Connect MQTT
- Disconnect MQTT
- MQTT Connected

Broker address, port and credentials are applied at runtime. A normal broker change therefore does not require a firmware rebuild.

The Topic Prefix field is currently reserved for a later custom UNS interface. It does not change ESPHome's automatic MQTT topic structure in the current firmware.

## Current MQTT interface

The current working interface is ESPHome's standard MQTT entity mapping.

Examples:

```text
andon-light-01/status

andon-light-01/select/andon_mode/command
andon-light-01/select/andon_mode/state

andon-light-01/switch/buzzer_mute_override/command
andon-light-01/switch/buzzer_mute_override/state

andon-light-01/button/acknowledge_alarm/command
```

Examples of command payloads:

```text
Andon Mode:
RUNNING
WARNING
FAULT
OFF

Buzzer Mute Override:
ON
OFF

Acknowledge Alarm:
PRESS
```

Home Assistant MQTT discovery is disabled because Home Assistant already uses the native ESPHome API for its normal entities.

See [docs/mqtt-uns.md](docs/mqtt-uns.md).

## UNS direction

A future version may expose a vendor-neutral semantic namespace such as:

```text
hupla/demo/factory01/line01/andon01/command/mode
hupla/demo/factory01/line01/andon01/state/mode
hupla/demo/factory01/line01/andon01/status/online
```

This is intentionally not implemented yet. The next UNS work should start from a concrete PLC, MES, SCADA or edge-data demo rather than adding a second MQTT API only for completeness.

## Repository structure

```text
.
├── README.md
├── ROADMAP.md
├── docs/
│   ├── alarm-philosophy.md
│   ├── architecture.md
│   ├── bring-up.md
│   ├── firmware.md
│   ├── hardware.md
│   ├── mqtt-uns.md
│   ├── status.md
│   └── update-model.md
└── esphome/
    ├── andon-light.yaml.example
    └── secrets.yaml.example
```

The deployed Home Assistant ESPHome file is normally:

```text
/config/esphome/andon-light-01.yaml
```

The repository file `esphome/andon-light.yaml.example` is the documented firmware baseline and must never contain real credentials.

## Next useful validation

1. Reboot or power-cycle and verify MQTT broker settings persist.
2. Change broker settings from the web UI and connect to a second broker without reflashing.
3. Test the Andon while Home Assistant is deliberately unavailable.
4. Verify Web OTA.
5. Finish physical protection with fuse, strain relief and enclosure.
6. Add custom UNS topics only when needed for a specific industrial integration demonstration.

See [ROADMAP.md](ROADMAP.md).
