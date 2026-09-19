# Update and provisioning model

Last reviewed: 2026-09-19

## Purpose

The device must remain usable in several situations:

1. normal development at home
2. commissioning on a new Wi-Fi network
3. connection to a new MQTT broker
4. standalone local control
5. recovery after network or firmware failure

The current design provides a separate path for each need.

## 1. UART serial flashing

Use for:

- first installation
- recovery when Wi-Fi is unavailable
- recovery when OTA is broken
- low-level troubleshooting

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

Normal firmware-development path.

Use it when changing:

- GPIO logic
- semantic Andon behavior
- MQTT implementation
- custom topic subscriptions/publications
- web-interface structure
- framework/components
- compiled defaults

Status: verified working.

## 3. Fallback AP and local web UI

If no known Wi-Fi network is available, the device exposes:

```text
Andon-Setup
```

Expected local address:

```text
http://192.168.4.1/
```

The project deliberately does not depend on ESPHome `captive_portal:`.

The normal Web Server is the fallback interface because commissioning also requires access to Andon controls and MQTT settings.

## 4. Runtime Wi-Fi provisioning

The web interface exposes:

- New WiFi SSID
- New WiFi Password
- Connect & Save WiFi

This uses `wifi.configure` with `save: true`.

A new Wi-Fi network therefore does not require a firmware rebuild.

## 5. Runtime MQTT provisioning

Firmware 0.5.0 exposes:

- MQTT Broker
- MQTT Port
- MQTT Username
- MQTT Password
- MQTT Topic Prefix
- Save & Connect MQTT
- Disconnect MQTT

Broker address, port and credentials can be changed without reflashing.

The MQTT values use ESPHome restore storage.

A deliberate full power-cycle persistence test is still outstanding.

### Topic Prefix limitation

The Topic Prefix value is stored, but custom Hupla/UNS topic logic is not implemented yet.

Changing Topic Prefix therefore does not currently change ESPHome's automatic MQTT topic names.

## 6. Local web interface

Normal-network access:

```text
http://andon-light-01.local/
```

Fallback access:

```text
http://192.168.4.1/
```

Current functions:

- semantic Andon control
- alarm acknowledge
- buzzer mute
- raw diagnostics
- Wi-Fi provisioning
- MQTT provisioning
- device/network status
- restart
- Web OTA component

Authentication is required.

The interface uses HTTP rather than HTTPS, so commissioning credentials should be used only on a trusted local network.

## 7. Web OTA

Web OTA is included as a field-maintenance path.

Workflow:

```text
build OTA firmware image
        |
open local Andon web UI
        |
upload OTA image
        |
device reboots
```

Use an OTA firmware image, not a factory image.

Status: component present, full browser OTA flow not yet deliberately verified.

## Decision guide

| Situation | Method |
| --- | --- |
| First flash | UART |
| Normal firmware change | ESPHome OTA |
| Unknown Wi-Fi | fallback AP + local web UI |
| Change Wi-Fi credentials | local web UI + `wifi.configure` |
| Change MQTT broker/IP | local web UI |
| Change MQTT port/login | local web UI |
| Change current standard ESPHome MQTT payload/state | MQTT client, no firmware update |
| Add custom MQTT/UNS topics | firmware update |
| Change flash/buzzer behavior | firmware update |
| Add TLS certificate logic | firmware update |
| Technician has only compiled OTA image | Web OTA |
| Wi-Fi/OTA broken | UART |

## Secrets model

ESPHome Device Builder secrets:

```text
/config/esphome/secrets.yaml
```

Home Assistant main secrets:

```text
/config/secrets.yaml
```

These are separate files.

Preserve the deployed API encryption key. It is generated cryptographic material, not a normal password.

Never store real credentials in this repository.

## Source of truth

Repository firmware baseline:

```text
esphome/andon-light.yaml.example
```

Live Device Builder YAML:

```text
/config/esphome/andon-light-01.yaml
```

The repository should describe the deployed baseline accurately, while secret values remain outside Git.
