# Architecture

Last reviewed: 2026-09-19

## Design objective

The Andon is an independent edge device, not a Home Assistant accessory.

The same firmware can currently provide:

1. local Andon logic
2. local browser control and commissioning
3. Home Assistant integration through the ESPHome native API
4. standard MQTT through a broker
5. runtime Wi-Fi provisioning
6. runtime MQTT broker provisioning
7. ESPHome OTA and UART recovery

A future UNS interface can be added without changing the basic hardware architecture.

## Current logical architecture

```text
                         ESP32 / ESPHome
                 local semantic Andon logic
                         4 MOSFET outputs
                              |
                            Wi-Fi
                              |
          +-------------------+-------------------+
          |                   |                   |
    local web UI       ESPHome native API        MQTT
          |                   |                   |
 commissioning          Home Assistant        Mosquitto
 direct control              |                   |
 Wi-Fi setup                 |          any MQTT subscriber
 MQTT setup                  |          or publisher
```

The device remains useful even if one control path is unavailable.

## Local behavior

The ESP32 owns the annunciation behavior.

External systems select a semantic mode. The ESP32 determines:

- physical color
- steady or flashing pattern
- buzzer pattern
- acknowledge behavior
- mute override

This prevents every upstream system from needing to understand GPIO numbers or timing.

See `docs/alarm-philosophy.md`.

## Interfaces

### Local web interface

Current role:

- mode control
- alarm acknowledge
- Buzzer Mute Override
- manual output diagnostics
- Wi-Fi provisioning
- MQTT broker provisioning
- MQTT connection status
- network/device diagnostics
- restart
- Web OTA component

The interface is authenticated with HTTP Basic Auth and is intended for trusted local or commissioning networks.

It is not a YAML editor.

### ESPHome native API

Current role:

- normal Home Assistant integration
- encrypted entity control
- status and diagnostics
- ESPHome development workflow

Home Assistant is optional for the MQTT/industrial use case.

### MQTT

Current role:

- broker-based control independent of Home Assistant's native API
- integration with other MQTT clients and tools
- portable connection to a different broker without firmware rebuild

Home Assistant MQTT discovery is disabled. Home Assistant uses the native ESPHome API for its regular entities.

Current working MQTT topics are ESPHome's standard entity topics, for example:

```text
andon-light-01/select/andon_mode/command
andon-light-01/select/andon_mode/state

andon-light-01/switch/buzzer_mute_override/command
andon-light-01/switch/buzzer_mute_override/state

andon-light-01/button/acknowledge_alarm/command

andon-light-01/status
```

The custom Hupla/UNS namespace is not active yet.

## Runtime provisioning

### Wi-Fi

The compiled configuration contains the initial home network.

When the device cannot join a known network, it exposes:

```text
Andon-Setup
```

The normal ESPHome web server remains available on the fallback AP. The project deliberately does not depend on `captive_portal:`.

A new SSID and password can be provided through `wifi.configure`.

### MQTT

Firmware 0.5.0 starts the MQTT component disabled, loads the saved runtime configuration and then enables it.

Runtime configurable fields:

- broker hostname or IP
- port
- username
- password
- topic prefix reserved for future custom MQTT/UNS logic

The broker can therefore be changed on an industrial site without recompiling the firmware.

## Home-lab reference configuration

Current tested flow:

```text
ESP32
  |
Wi-Fi
  |
192.168.129.15:1883
  |
Mosquitto
  |
Home Assistant MQTT listen/publish
```

This home broker is a test environment, not an architectural dependency.

## Industrial-site flow

Target current workflow:

```text
bring Andon to site
      |
configure site Wi-Fi
      |
open local Andon web UI
      |
enter site MQTT broker / port / credentials
      |
Save & Connect MQTT
      |
test standard MQTT topics
```

No firmware rebuild is required for a normal TCP broker with username/password.

Customer-specific TLS certificates or mutual TLS are not runtime-provisioned in the current design.

## Future UNS architecture

A later experiment may decouple the Andon from dedicated device commands.

Example:

```text
PLC / MES / SCADA / edge application
                |
        OPC UA / MQTT gateway
                |
              UNS
                |
        +-------+-------+
        |       |       |
       MES   dashboard  Andon
```

The Andon could then consume operational meaning such as `FAULT` rather than receiving a raw request to switch a lamp.

Possible future namespace:

```text
hupla/demo/factory01/line01/andon01/
```

This is a design direction only. It is deliberately not part of firmware 0.5.0.

## Failure behavior

Current design choices:

- `api.reboot_timeout: 0s` so loss of Home Assistant does not reboot the device
- `mqtt.reboot_timeout: 0s` so loss of the broker does not reboot the device
- raw GPIO outputs use `restore_mode: ALWAYS_OFF`
- semantic behavior is calculated locally
- MQTT can be disconnected or unavailable without removing local web control

Still to verify explicitly:

- behavior through a full power-cycle test
- operation while Home Assistant is deliberately stopped
- operation against a second broker
- reconnect behavior after a real network interruption

## Source of truth

Firmware baseline:

```text
esphome/andon-light.yaml.example
```

Live Device Builder configuration normally lives at:

```text
/config/esphome/andon-light-01.yaml
```

Real credentials remain only in ESPHome secrets and must never be committed.

## References

- ESPHome MQTT: https://esphome.io/components/mqtt/
- ESPHome Wi-Fi: https://esphome.io/components/wifi/
- ESPHome Web Server: https://esphome.io/components/web_server/
