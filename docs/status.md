# Project status - 2026-09-19

## Working now

- ESP32-WROOM-32E board received
- UART header soldered
- Silicon Labs CP210x USB-UART adapter working on Windows
- first ESPHome flash completed
- ESPHome 2026.8.2 running
- ESP32 rev 3.1 identified
- Wi-Fi connection working
- encrypted ESPHome native API working
- OTA update working
- OTA verified by changing the friendly name and uploading wirelessly
- ESPHome Web OTA component present in runtime

## Not connected yet

- 12 V Andon
- MQTT broker
- UNS namespace

## Hardware checks still required

Expected MOSFET mapping:

| Output | Expected GPIO | Planned load |
| --- | ---: | --- |
| OUT1 | GPIO16 | Red |
| OUT2 | GPIO17 | Yellow |
| OUT3 | GPIO26 | Green |
| OUT4 | GPIO27 | Buzzer |

This mapping is still unverified on the actual board.

The next physical test is to run from the board's 12 V DC input and verify that:

1. the ESP32 boots from that one supply
2. each GPIO drives the expected MOSFET channel
3. outputs remain off during boot
4. the Andon can then be attached one channel at a time

## Latest fallback-interface decision

The goal is a standalone Andon controller that remains usable without Home Assistant or an existing LAN.

Next firmware target:

```text
Normal network available
    |
    +--> ESPHome API --> Home Assistant
    +--> local web UI
    +--> later MQTT / UNS

No known network available
    |
    +--> Andon-Setup fallback AP
           |
           +--> local web UI at 192.168.4.1
                  |
                  +--> Red / Yellow / Green / Buzzer
                  +--> New WiFi SSID
                  +--> New WiFi Password
                  +--> Connect & Save WiFi
                  +--> Web OTA
```

The project therefore moves away from using the captive portal as the main fallback UI. The normal ESPHome Web Server will be kept available on the fallback AP, and `wifi.configure` will be used for runtime Wi-Fi provisioning.

## Secrets

Current API logs confirm encryption is already active. Preserve the existing generated API encryption key.

ESPHome Device Builder normally uses:

```text
/config/esphome/secrets.yaml
```

This is separate from:

```text
/config/secrets.yaml
```

used by Home Assistant itself.
