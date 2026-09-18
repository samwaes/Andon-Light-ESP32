# Firmware strategy

## Platform

ESPHome is the firmware framework.

The project deliberately separates:

- firmware framework: ESPHome
- Home Assistant integration: ESPHome native API
- industrial integration: MQTT
- physical control: local ESPHome logic

This allows one ESP32 to work both at home and in an industrial demo without making Home Assistant mandatory.

## Connectivity stack

Target configuration:

```yaml
api:
  encryption:
    key: !secret api_encryption_key

mqtt:
  broker: !secret mqtt_broker
  username: !secret mqtt_username
  password: !secret mqtt_password
  discovery: false
  discover_ip: true
```

ESPHome currently documents using the native API and MQTT together. When Home Assistant connects over the native API, MQTT entity discovery can be disabled to avoid duplicate entities while MQTT remains available for industrial/UNS messaging.

## Wi-Fi

Use multiple known networks:

```yaml
wifi:
  networks:
    - ssid: !secret wifi_home_ssid
      password: !secret wifi_home_password

    - ssid: !secret wifi_demo_ssid
      password: !secret wifi_demo_password

  ap:
    ssid: !secret fallback_ap_ssid
    password: !secret fallback_ap_password

captive_portal:
```

Do not hard-code credentials in the main YAML committed to GitHub.

## Framework choice

Start with the Arduino framework for the first MQTT/ESPHome implementation. MQTT under ESP-IDF is currently documented by ESPHome as experimental.

This can be reconsidered later if another ESP-IDF-specific feature becomes necessary.

## OTA

After the first UART flash, normal firmware updates should happen over Wi-Fi.

Two OTA paths are planned:

1. ESPHome OTA for normal development from the YAML source.
2. Web-server OTA for field updates using a precompiled firmware image.

UART is retained as the recovery method if networking or OTA becomes unusable.

The normal web interface does not edit the YAML source. Configuration changes such as GPIO logic, MQTT behavior or known SSIDs still belong in the repository and require a firmware rebuild. Wi-Fi credentials are the exception because the captive portal can provision a network at runtime.

## Web interface

A local ESPHome web server may be added for commissioning and demonstrations.

The web server is enabled for commissioning and demonstrations.

Requirements:

- require authentication
- prefer local embedded assets if offline operation is needed
- do not treat it as the main industrial interface
- avoid exposing it directly to untrusted networks
- enable web OTA so a compiled firmware image can be installed without ESPHome tooling

## State model

The firmware should eventually provide two layers.

### Raw outputs

Four direct switches for commissioning:

- output 1
- output 2
- output 3
- output 4

These remain diagnostic entities.

### Semantic Andon mode

Normal control uses a mode:

```text
OFF
RUNNING
WARNING
FAULT
STOPPED
MAINTENANCE
```

Changing a mode applies all four physical outputs together.

## Boot behavior

Requirements:

1. All outputs start in a known state.
2. No unintended lamp or buzzer pulse occurs during reset.
3. Last state restoration is not enabled until output boot behavior has been verified.
4. Initial development should default to all outputs OFF.

## MQTT behavior

The firmware should:

- connect to a standard MQTT broker
- publish availability
- subscribe to semantic command topics
- publish current state as retained messages
- publish events as non-retained messages
- republish state after reconnect

## GPIO mapping

Not yet known.

Do not copy arbitrary ESP32 GPIO examples into the live configuration. The quad-MOS board has its own hardwired GPIO-to-MOSFET mapping.

The first test firmware should identify one channel at a time and update the documentation when confirmed.

## References

- ESPHome MQTT: https://esphome.io/components/mqtt/
- ESPHome Wi-Fi: https://esphome.io/components/wifi/
- ESPHome Web Server: https://esphome.io/components/web_server/
