# Firmware strategy

Last reviewed: 2026-09-19

## Current firmware

Current deployed baseline:

```text
0.5.0
```

Platform:

- ESPHome 2026.8.2
- ESP32-WROOM-32E
- ESP-IDF framework
- encrypted ESPHome native API
- ESPHome OTA
- Web Server v3
- standard MQTT
- local semantic Andon logic

Canonical repository baseline:

```text
esphome/andon-light.yaml.example
```

Live Home Assistant Device Builder file normally resides at:

```text
/config/esphome/andon-light-01.yaml
```

## Design separation

The firmware deliberately separates:

- physical GPIO outputs
- semantic Andon behavior
- local web commissioning
- Home Assistant native API
- MQTT transport
- future custom UNS semantics

This keeps the device useful even when Home Assistant is not part of the industrial environment.

## GPIO model

Confirmed mapping:

| Output | GPIO | Function |
| --- | ---: | --- |
| OUT1 | GPIO16 | Red |
| OUT2 | GPIO17 | Yellow |
| OUT3 | GPIO26 | Green |
| OUT4 | GPIO27 | Buzzer |

Raw GPIO switches use:

```yaml
restore_mode: ALWAYS_OFF
```

They remain visible for manual diagnostics.

## Semantic state model

Current mode options:

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

The `render_andon` script runs from a 250 ms timing base and calculates:

- lamp color
- flash state
- buzzer state
- acknowledge behavior
- mute override

The TD-50 red/yellow hardware limitation is handled by using one semantic lamp color at a time.

See `docs/alarm-philosophy.md`.

## Home Assistant API

Current configuration intentionally includes:

```yaml
api:
  encryption:
    key: !secret api_encryption_key
  reboot_timeout: 0s
```

The encryption key must be preserved when replacing the live YAML.

`reboot_timeout: 0s` prevents loss of Home Assistant from forcing an ESP32 reboot.

## Wi-Fi and fallback

Current baseline:

```yaml
wifi:
  networks:
    - ssid: !secret wifi_home_ssid
      password: !secret wifi_home_password

  ap:
    ssid: "Andon-Setup"
    password: !secret fallback_ap_password
    ap_timeout: 20s

  reboot_timeout: 0s
  power_save_mode: none
```

There is deliberately no `captive_portal:`.

The normal Web Server remains available while connected to the fallback AP.

## Runtime Wi-Fi configuration

The web interface exposes:

- New WiFi SSID
- New WiFi Password
- Connect & Save WiFi

The button uses `wifi.configure` with `save: true`.

This allows network commissioning without UART reflashing.

## Local web interface

Current groups:

1. Andon Mode & Alarm
2. Manual / Diagnostics
3. WiFi Setup
4. MQTT Setup
5. Device Status
6. System

Web Server v3 uses embedded local assets so the page remains usable when the fallback AP has no Internet route.

The interface uses HTTP Basic Auth. Treat Wi-Fi and MQTT credentials entered through it as commissioning credentials on a trusted local network.

## Alarm entities

Primary semantic entities:

- Andon Mode
- Buzzer Mute Override
- Acknowledge Alarm
- Clear / OFF
- Alarm Acknowledged

Diagnostic entities:

- Manual Red
- Manual Yellow
- Manual Green
- Manual Buzzer

## Runtime MQTT provisioning

Firmware 0.5.0 no longer requires the MQTT broker address to be fixed at compile time.

The MQTT component is declared disabled at boot:

```yaml
mqtt:
  id: mqtt_client
  broker: ""
  port: 1883
  enable_on_boot: false
  discovery: false
  reboot_timeout: 0s
```

On boot the firmware reads the restored runtime fields, applies them to the MQTT client and enables MQTT.

Runtime fields:

- MQTT Broker
- MQTT Port
- MQTT Username
- MQTT Password
- MQTT Topic Prefix

Save & Connect MQTT:

1. disables the active MQTT session
2. applies broker/port/credentials
3. enables MQTT again

The broker can therefore be changed without a firmware rebuild.

### Persistence status

The firmware uses `restore_value: true` for the runtime MQTT fields.

The design therefore persists values in ESP flash, but a deliberate full power-cycle verification remains on the roadmap.

## Current MQTT topics

The current working MQTT API is ESPHome's standard entity topic model.

Examples:

```text
andon-light-01/status

andon-light-01/select/andon_mode/command
andon-light-01/select/andon_mode/state

andon-light-01/switch/buzzer_mute_override/command
andon-light-01/switch/buzzer_mute_override/state

andon-light-01/button/acknowledge_alarm/command
```

Examples:

```text
publish FAULT to:
andon-light-01/select/andon_mode/command

publish ON to:
andon-light-01/switch/buzzer_mute_override/command

publish PRESS to:
andon-light-01/button/acknowledge_alarm/command
```

This has been validated through Home Assistant's MQTT listen/publish tools.

MQTT discovery remains disabled to avoid duplicate Home Assistant entities because the native API is already present.

## MQTT Topic Prefix field

Firmware 0.5.0 stores:

```text
hupla/demo/factory01/line01/andon01
```

as the initial Topic Prefix value.

Important: this field is currently reserved for later custom MQTT/UNS logic. It does not modify the standard ESPHome topic hierarchy in firmware 0.5.0.

Do not describe the custom Hupla topic hierarchy as active until the custom subscriptions/publications are implemented and tested.

## TLS scope

Current runtime provisioning covers ordinary MQTT TCP with username/password.

Customer-specific runtime provisioning of:

- CA certificate
- client certificate
- client private key
- mutual TLS

is not implemented.

## OTA and recovery

Three firmware paths remain:

1. ESPHome OTA for normal development
2. Web OTA for field update using a compiled OTA binary
3. UART serial flashing for first install and recovery

ESPHome OTA is verified.

Web OTA is present but still requires a deliberate end-to-end test.

UART was used successfully for the first flash.

## Secrets

Device Builder normally reads:

```text
/config/esphome/secrets.yaml
```

This is separate from Home Assistant's main:

```text
/config/secrets.yaml
```

Never commit real credentials.

Preserve the existing generated API encryption key and deployed passwords unless intentionally rotating them.

See `esphome/secrets.yaml.example`.

## Future firmware work

Not required for the current baseline:

- custom semantic MQTT/UNS topic subscriptions
- retained custom state
- custom birth/last-will topic
- non-retained event stream
- dynamic TLS certificate provisioning
- physical acknowledge button
- Ethernet variant

These should be added only when a concrete demonstration requires them.
