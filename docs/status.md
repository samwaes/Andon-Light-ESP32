# Project status - 2026-09-19

## Working now

- ESP32-WROOM-32E controller operational
- HNTD TD-50 Andon powered from the 12 V setup
- GPIO16 / GPIO17 / GPIO26 / GPIO27 mapped to red / yellow / green / buzzer
- Wi-Fi and encrypted ESPHome native API working
- ESPHome OTA working
- local ESPHome web portal tested successfully
- fallback/local web architecture implemented
- semantic modes, flash patterns, alarm acknowledge and Buzzer Mute Override working
- TD-50 internal red/yellow interaction confirmed as an Andon hardware limitation

## Firmware 0.5.0

Firmware 0.5.0 is deployed and working.

The local web portal now allows runtime MQTT configuration of:

- broker hostname or IP address
- port
- username
- password
- topic prefix
- Save & Connect MQTT
- Disconnect MQTT
- MQTT Connected status

The intention is field portability: at an industrial site the Andon can join the local Wi-Fi and be pointed at that site's MQTT broker without recompiling or reflashing firmware.

## MQTT validation

Home Assistant Mosquitto is installed and running at:

```text
192.168.129.15:1883
```

The Andon connects successfully with its dedicated MQTT credentials.

Verified on 2026-09-19:

- MQTT connection succeeds
- Home Assistant can listen to Andon MQTT traffic
- Home Assistant can publish MQTT commands
- Andon mode can be changed over the standard ESPHome MQTT command topic
- current state is published back over the corresponding ESPHome state topic
- Buzzer Mute Override can be controlled over MQTT
- Acknowledge can be triggered over MQTT
- Home Assistant MQTT discovery remains disabled to avoid duplicate entities beside the native ESPHome API

The current working interface is the standard ESPHome MQTT topic structure. The planned custom `hupla/demo/factory01/...` semantic UNS topic structure is deliberately deferred.

## Current project position

The demonstrator now proves three independent control paths:

```text
Local Andon web UI
        |
Home Assistant native ESPHome API
        |
Standard MQTT through Mosquitto
        |
      ESP32
        |
   Andon tower
```

This is sufficient as the present baseline. The next UNS layer should only be added when it supports a concrete demo scenario rather than as extra protocol work by itself.

## Later validation

1. Verify persisted MQTT settings after a full power cycle.
2. Change broker configuration from the web portal and connect to a second broker without reflashing.
3. Validate operation while Home Assistant is unavailable.
4. Verify browser-based web OTA.
5. Later decide on custom semantic topics, retained state, birth/last-will and a fuller UNS demo.
