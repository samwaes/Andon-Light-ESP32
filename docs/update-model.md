# Update and provisioning model

## Purpose

The Andon must remain maintainable in several situations:

1. Development at home
2. Use in an industrial lab on an unfamiliar Wi-Fi network
3. Direct standalone use without Home Assistant
4. Recovery after a configuration or network failure

The design therefore uses several complementary mechanisms.

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

Status: first serial flash completed successfully.

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
- updating the local web interface

Status: verified working by changing the friendly name and updating wirelessly.

The YAML in this repository remains the source of truth.

## 3. Fallback Wi-Fi AP

Known Wi-Fi SSIDs can be compiled into the firmware.

If none are available, the ESP32 should start a protected fallback access point:

```text
Andon-Setup
```

Target fallback address:

```text
http://192.168.4.1/
```

### Design change

The project will not use ESPHome's captive portal as the main fallback interface.

Reason: the fallback interface must do more than provision Wi-Fi. It must also allow direct control of:

- Red
- Yellow
- Green
- Buzzer

The normal ESPHome Web Server will therefore remain the fallback UI.

## 4. Runtime Wi-Fi provisioning

The local web interface will expose:

- New WiFi SSID
- New WiFi Password
- Connect & Save WiFi

The button uses ESPHome `wifi.configure`.

Concept:

```yaml
- wifi.configure:
    ssid: !lambda 'return id(setup_wifi_ssid).state;'
    password: !lambda 'return id(setup_wifi_password).state;'
    save: true
    timeout: 30s
```

With `save: true`, the credentials are persisted by ESPHome.

This allows the Andon to be moved to a lab network without UART reflashing.

## 5. Local web interface

The ESPHome Web Server provides:

- direct Andon output controls
- Wi-Fi provisioning controls
- device/network status
- restart control
- browser-based OTA

Use:

```text
http://andon-light-01.local/
```

on a normal network, or:

```text
http://192.168.4.1/
```

on the fallback AP.

Use `local: true` so the interface assets are embedded in the ESP32 and remain available with no Internet route.

Authentication is required.

## 6. Web OTA

The local web interface also exposes browser-based OTA.

Workflow:

```text
Build firmware.ota.bin
        |
Open Andon web interface
        |
Upload firmware image
        |
ESP32 reboots into new firmware
```

This is useful when the operator has a firmware image but no ESPHome development environment.

Use an OTA firmware image for web OTA, not a factory image.

Status: component is present in the current runtime, but an actual browser OTA update is not yet verified.

## Decision guide

| Situation | Recommended method |
| --- | --- |
| First ever flash | UART |
| Normal firmware development | ESPHome OTA |
| New/unknown Wi-Fi | Fallback AP + local web UI + wifi.configure |
| Standalone control with no LAN | Fallback AP + local web UI |
| Technician has only OTA firmware image | Web OTA |
| Wi-Fi/OTA is broken | UART |
| Change GPIO/MQTT logic | ESPHome OTA or Web OTA with rebuilt firmware |
| Change only Wi-Fi network | Local web UI + wifi.configure |

## Secrets model

ESPHome Device Builder normally uses:

```text
/config/esphome/secrets.yaml
```

This is separate from Home Assistant's main:

```text
/config/secrets.yaml
```

The current device already has API encryption enabled. Preserve the existing API encryption key when replacing the YAML.

The API encryption key is a generated cryptographic key and should not be replaced by an arbitrary password string.

## MQTT broker portability

Wi-Fi provisioning does not automatically solve MQTT broker discovery.

The MQTT broker hostname remains a separate design problem. The preferred demo architecture is to keep a stable broker identity, for example through a portable demo network or consistent DNS name.

MQTT will be added after the physical outputs and fallback web interface are verified.
