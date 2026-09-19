# Project status

Last reviewed: 2026-09-19

## Overall status

Working MQTT demonstrator.

Firmware 0.5.0 is deployed on the physical ESP32/Andon assembly.

The project has moved beyond hardware bring-up. The current priority is to preserve a stable working baseline and validate portability before adding any custom UNS layer.

## Working

### Hardware

- ESP32-WROOM-32E quad-MOSFET controller
- 12 V HNTD TD-50 Andon
- red, yellow, green and buzzer outputs
- common 12 V power arrangement
- UART recovery hardware

Confirmed mapping:

```text
GPIO16 -> OUT1 -> Red
GPIO17 -> OUT2 -> Yellow
GPIO26 -> OUT3 -> Green
GPIO27 -> OUT4 -> Buzzer
```

### Local behavior

- semantic Andon modes
- flash patterns
- buzzer patterns
- acknowledge
- Clear / OFF
- Buzzer Mute Override
- manual diagnostic outputs

### Connectivity and management

- home Wi-Fi
- protected fallback AP
- local Web Server v3
- runtime Wi-Fi provisioning
- encrypted ESPHome native API
- ESPHome OTA
- runtime MQTT broker configuration

### MQTT

Home-lab Mosquitto:

```text
192.168.129.15:1883
```

Verified:

- MQTT connects
- Home Assistant listens successfully
- Home Assistant publishes successfully
- Andon mode can be changed through MQTT
- mode state is published
- buzzer mute can be controlled
- acknowledge can be triggered
- Home Assistant MQTT discovery is disabled to prevent duplicate entities

Current MQTT API:

```text
standard ESPHome MQTT topics
```

Custom Hupla/UNS topics are not active.

## Known limitation

The TD-50 has an internal red/yellow interaction.

Red and yellow cannot be relied on simultaneously even after swapping the control wires across different MOSFET outputs.

Operational consequence:

- semantic modes use one color at a time
- multi-color combinations are diagnostics only

## Runtime MQTT configuration

Available through the web UI:

- broker hostname/IP
- port
- username
- password
- topic prefix
- Save & Connect
- Disconnect
- connected status

The Topic Prefix field is reserved for later custom UNS logic and does not currently change standard ESPHome topics.

## Open validation

- full power-cycle persistence
- second MQTT broker
- Home Assistant deliberately offline
- runtime Wi-Fi persistence after full power loss
- Web OTA
- safe boot/output observation
- final fuse/strain relief/enclosure

## Deferred work

Not required for the current baseline:

- custom semantic MQTT command/state topics
- custom retained state
- custom birth/last-will
- event stream
- OPC UA bridge
- Sparkplug B
- Node-RED demo
- MES/SCADA scenario
- runtime TLS certificate provisioning

## Current project decision

Do not extend MQTT only for architectural neatness.

The next custom MQTT/UNS work should be tied to a concrete industrial scenario so the Andon demonstrates producer/consumer decoupling rather than just another topic hierarchy.
