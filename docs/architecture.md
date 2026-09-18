# Architecture

## Design objective

The Andon should not be a Home Assistant accessory that happens to speak MQTT. It should be an independent edge device that can participate in multiple environments.

The same ESP32 firmware should support:

1. Local output logic
2. Home Assistant through the ESPHome native API
3. Standard MQTT through a broker
4. Optional browser access for commissioning
5. OTA firmware updates

## Logical architecture

```text
                         +----------------------+
                         | ESP32 / ESPHome      |
                         |                      |
                         | local state logic    |
                         | 4 MOSFET outputs     |
                         +----------+-----------+
                                    |
                                  Wi-Fi
                                    |
              +---------------------+---------------------+
              |                     |                     |
              v                     v                     v
      ESPHome native API          MQTT             local web UI
              |                     |
              v                     v
       Home Assistant          MQTT broker
                                    |
                       +------------+------------+
                       |            |            |
                       v            v            v
                    Node-RED      UNS        MQTT Explorer
```

## Interface separation

### ESPHome native API

Purpose:

- best Home Assistant integration
- device entities
- logs and diagnostics
- encrypted control
- OTA workflow

Home Assistant is optional. The ESP32 should not require an active Home Assistant connection to perform its basic Andon function.

### MQTT

Purpose:

- portable industrial interface
- integration with Node-RED, MQTT Explorer, test software, simulators and UNS platforms
- operation without Home Assistant

ESPHome supports MQTT and the native API together. For this project the native API is the Home Assistant control path and custom MQTT topics are the industrial/UNS control path.

### Local logic

The device should expose semantic modes instead of forcing every client to understand raw outputs.

Initial modes:

| Mode | Red | Yellow | Green | Buzzer |
| --- | ---: | ---: | ---: | ---: |
| OFF | 0 | 0 | 0 | 0 |
| RUNNING | 0 | 0 | 1 | 0 |
| WARNING | 0 | 1 | 0 | 0 |
| FAULT | 1 | 0 | 0 | 1 |
| STOPPED | 1 | 0 | 0 | 0 |
| MAINTENANCE | 0 | 1 | 0 | 0 |

Raw output control should remain available for diagnostics, but normal applications should use the mode abstraction.

## Home network

Typical flow:

```text
ESP32 -> home Wi-Fi -> Home Assistant native API
                  \
                   -> MQTT broker -> MQTT/UNS tools
```

## Industrial lab

Typical flow:

```text
ESP32 -> lab Wi-Fi -> MQTT broker -> Node-RED / UNS / dashboard
```

Home Assistant is not required in this mode.

## Network portability

Wi-Fi and MQTT broker discovery are separate problems.

ESPHome supports multiple configured Wi-Fi networks, so one firmware can know the home SSID and one or more lab/demo SSIDs.

The MQTT broker address is normally configured as one hostname/IP. Three approaches are possible:

### A. Portable demo network

Preferred for demonstrations.

Use a dedicated travel router or demo access point and always provide the same broker hostname/IP.

Advantages:

- predictable
- independent of customer IT
- works offline
- easier troubleshooting

### B. Common DNS hostname

Use the same MQTT broker DNS name in every environment and resolve it appropriately.

Advantages:

- same firmware
- no local reconfiguration

Constraint: requires DNS/network control.

### C. Fixed remote broker

Use an Internet-reachable MQTT broker.

Advantages:

- same broker everywhere

Constraints:

- depends on Internet
- industrial networks may block outbound MQTT
- requires proper TLS and credentials

For the first lab implementation, option A is the target.

## Failure behavior

The device should be designed so that:

- loss of Home Assistant does not change the current Andon state
- loss of MQTT does not reboot the device
- Wi-Fi loss does not cause outputs to flicker
- boot starts in a known safe state
- reconnect publishes current state again
- MQTT availability reports online/offline through birth and last-will messages

## References

- ESPHome Wi-Fi: https://esphome.io/components/wifi/
- ESPHome MQTT: https://esphome.io/components/mqtt/
- ESPHome Web Server: https://esphome.io/components/web_server/
