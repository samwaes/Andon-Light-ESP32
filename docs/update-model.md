# Update and provisioning model

## Purpose

The Andon should remain maintainable in three very different situations:

1. Development at home
2. Use in an industrial lab on an unfamiliar Wi-Fi network
3. Recovery after a configuration or network failure

No single update mechanism is ideal for all three situations, so the firmware deliberately supports several.

## 1. UART serial flashing

Use for:

- first installation
- recovery when Wi-Fi is unavailable
- recovery when OTA is broken
- low-level debugging

Connection:

```text
USB-UART       ESP32 board
TXD         -> RX
RXD         -> TX
GND         -> GND
```

IO0 is held low during boot to enter the ESP32 bootloader.

UART is the lowest-level recovery path and should always remain documented even when normal updates happen wirelessly.

## 2. ESPHome OTA

This is the normal development method.

Workflow:

```text
Edit YAML
   |
Compile with ESPHome
   |
Upload over Wi-Fi
   |
ESP32 reboots into new firmware
```

Use this for:

- GPIO changes
- MQTT changes
- adding/removing features
- changing known Wi-Fi networks
- changing local logic
- updating the web interface configuration

The YAML in this repository is the source of truth.

## 3. Fallback Wi-Fi and captive portal

Known Wi-Fi SSIDs are normally compiled into the firmware.

If none are available, the ESP32 starts a protected fallback access point:

```text
Andon-Setup
```

A phone or laptop can connect to this AP and use the ESPHome captive portal to provision another Wi-Fi network.

This is intended for moving the Andon into a new lab or demo network without reflashing it over UART.

Important distinction:

- captive portal changes the network the device connects to
- captive portal does not edit the complete ESPHome YAML

## 4. Local web interface

The ESPHome web server provides local status, diagnostics and control.

Planned URL on networks with mDNS support:

```text
http://andon-light-01.local
```

The interface requires authentication.

It is not a full firmware configuration editor.

## 5. Web OTA

The web interface also exposes browser-based OTA.

Workflow:

```text
Build firmware.bin elsewhere
        |
Open Andon web interface
        |
Upload firmware image
        |
ESP32 reboots into new firmware
```

This is useful in a lab where the operator has a firmware file but does not have an ESPHome development environment.

## Decision guide

| Situation | Recommended method |
| --- | --- |
| First ever flash | UART |
| Normal firmware development | ESPHome OTA |
| New/unknown Wi-Fi | Captive portal |
| Technician has only firmware.bin | Web OTA |
| Wi-Fi/OTA is broken | UART |
| Change GPIO/MQTT logic | ESPHome OTA or web OTA with rebuilt firmware |
| Change only Wi-Fi network | Captive portal |

## MQTT broker portability

Wi-Fi provisioning does not automatically solve MQTT broker discovery.

The MQTT broker hostname remains part of the ESPHome configuration. The preferred demo architecture is therefore to keep a stable broker identity, for example through a portable demo network or consistent DNS name.

This avoids changing firmware simply because the Andon is moved between locations.
