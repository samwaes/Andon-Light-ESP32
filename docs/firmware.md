# Firmware strategy

## Platform

ESPHome is the firmware framework.

Current bring-up status:

- first flash successful
- ESPHome 2026.8.2 running
- ESP32 rev 3.1 detected
- encrypted native API operational
- OTA update operational
- device online over Wi-Fi

The project deliberately separates:

- firmware framework: ESPHome
- Home Assistant integration: ESPHome native API
- industrial integration: MQTT
- physical control: local ESPHome logic
- fallback commissioning/control: local ESPHome web server

This allows one ESP32 to work both at home and in an industrial demo without making Home Assistant mandatory.

## Home Assistant API

The current device already uses API encryption. Preserve the existing generated API key when changing the configuration.

Target:

```yaml
api:
  encryption:
    key: !secret api_encryption_key
  reboot_timeout: 0s
```

`reboot_timeout: 0s` is intentional for the standalone design. Loss of Home Assistant must not cause the Andon controller to reboot.

## Wi-Fi

The device should support one or more known infrastructure networks plus a protected fallback AP.

```yaml
wifi:
  networks:
    - ssid: !secret wifi_home_ssid
      password: !secret wifi_home_password

    - ssid: !secret wifi_demo_ssid
      password: !secret wifi_demo_password

  ap:
    ssid: "Andon-Setup"
    password: !secret fallback_ap_password
    ap_timeout: 20s

  reboot_timeout: 0s
```

### Final fallback design decision

Do not use `captive_portal:` as the primary fallback UI.

Reason: the requirement is not only to provision Wi-Fi, but also to keep direct control of Red, Yellow, Green and Buzzer available when the ESP32 is running only as its own access point.

The next firmware baseline therefore uses:

- fallback AP
- normal ESPHome Web Server
- Web Server v3 control entities
- text fields for new SSID and password
- `wifi.configure` to connect and persist the new Wi-Fi credentials

Expected fallback workflow:

```text
No known Wi-Fi
   |
Andon-Setup appears
   |
connect phone/laptop
   |
http://192.168.4.1/
   |
control Andon directly
and/or enter new Wi-Fi
```

## Local web interface

Use Web Server v3 with local assets:

```yaml
web_server:
  port: 80
  version: 3
  local: true
  auth:
    username: !secret web_username
    password: !secret web_password
```

The local assets are important because the fallback AP may have no Internet route.

Planned local UI groups:

1. Andon Controls
2. WiFi Setup
3. Device Status
4. System

The web interface is not a YAML editor. Firmware behavior still comes from this repository.

## Runtime Wi-Fi configuration

ESPHome `wifi.configure` can accept templated SSID and password values and save them persistently.

Planned action:

```yaml
- wifi.configure:
    ssid: !lambda 'return id(setup_wifi_ssid).state;'
    password: !lambda 'return id(setup_wifi_password).state;'
    save: true
    timeout: 30s
```

This is intended for demos and commissioning when the Andon is moved to a new network.

## OTA

Three update/recovery paths are retained:

1. ESPHome OTA for normal development
2. Web Server OTA for browser-based field updates
3. UART serial flashing for first install and recovery

The normal development path is now confirmed working.

For browser OTA, use an OTA firmware binary, not a factory image.

## Framework choice

Keep the framework used by the current working ESPHome device unless there is a reason to change it. Do not change framework and functional behavior in the same troubleshooting step.

The example configuration currently uses ESP-IDF, matching the current ESPHome ESP32 baseline.

## GPIO / output model

Expected board-family mapping:

| Output | Expected GPIO | Planned function |
| --- | ---: | --- |
| OUT1 | GPIO16 | Red |
| OUT2 | GPIO17 | Yellow |
| OUT3 | GPIO26 | Green |
| OUT4 | GPIO27 | Buzzer |

This is not yet physically confirmed.

Each output should initially use:

```yaml
restore_mode: ALWAYS_OFF
```

so a normal reboot starts with the Andon outputs off.

## State model

The firmware will provide two layers.

### Raw outputs

Four direct switches for commissioning:

- Red
- Yellow
- Green
- Buzzer

These should be available through both the Home Assistant API and the local web server.

### Semantic Andon mode

Normal control will later use a mode:

```text
OFF
RUNNING
WARNING
FAULT
STOPPED
MAINTENANCE
```

Changing a mode will apply all four physical outputs together.

## MQTT

MQTT is deliberately postponed until the physical outputs and fallback interface are verified.

Later the firmware should:

- connect to a standard MQTT broker
- keep Home Assistant on the native ESPHome API
- disable duplicate Home Assistant MQTT entity discovery
- publish availability
- subscribe to semantic command topics
- publish current state as retained messages
- publish events as non-retained messages
- republish state after reconnect

## Secrets

The ESPHome secrets file used by Device Builder is normally:

```text
/config/esphome/secrets.yaml
```

This is separate from Home Assistant's main:

```text
/config/secrets.yaml
```

The existing API encryption key should be preserved. It is a generated cryptographic key, not an arbitrary human password.

The fallback AP password, web username/password and OTA password can be chosen, but existing values should generally be retained when already deployed to avoid breaking connectivity.

## References

- ESPHome MQTT: https://esphome.io/components/mqtt/
- ESPHome Wi-Fi: https://esphome.io/components/wifi/
- ESPHome Web Server: https://esphome.io/components/web_server/
- ESPHome Web OTA: https://esphome.io/components/ota/web_server/


## Alarm annunciation layer

Firmware version 0.4.0 adds a semantic annunciation layer above the four raw outputs.

Primary entities exposed to both the ESPHome web interface and Home Assistant:

- Andon Mode
- Buzzer Mute Override
- Acknowledge Alarm
- Clear / OFF
- Alarm Acknowledged

The raw Red, Yellow, Green and Buzzer switches remain available in a separate Manual / Diagnostics section.

The renderer runs every 250 ms and derives the physical outputs from the selected mode, flash phase, acknowledge state and mute override.

The buzzer mute override has the highest priority over audible behavior. It never changes the selected Andon mode and never acknowledges an alarm.

See `docs/alarm-philosophy.md` for the complete mode table and timing model.
