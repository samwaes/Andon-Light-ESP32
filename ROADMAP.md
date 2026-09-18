# Roadmap

## Phase 0 - Project baseline

Status: in progress

- [x] Select 12 V HNTD TD-50 Andon light
- [x] Select ESP32 quad MOSFET controller
- [x] Receive USB-to-UART adapter
- [x] Define Home Assistant + MQTT + UNS architecture
- [x] Create project repository and baseline documentation
- [ ] Photograph and archive the final hardware assembly
- [ ] Record exact part/vendor references

## Phase 1 - ESP32 bring-up

- [ ] Solder the six-pin UART programming header
- [ ] Confirm USB-UART logic level is 3.3 V
- [ ] Connect TXD -> RX, RXD -> TX, GND -> GND
- [ ] Power the ESP32 board independently
- [ ] Enter bootloader mode using IO0
- [ ] Flash minimal ESPHome firmware
- [ ] Verify serial log output
- [ ] Join home Wi-Fi
- [ ] Verify OTA update

Success criterion: the ESP32 can be reflashed over Wi-Fi without the UART adapter.

## Phase 2 - MOSFET output mapping

- [ ] Determine the ESP32 GPIO for OUT1
- [ ] Determine the ESP32 GPIO for OUT2
- [ ] Determine the ESP32 GPIO for OUT3
- [ ] Determine the ESP32 GPIO for OUT4
- [ ] Confirm whether GPIO high means MOSFET ON
- [ ] Confirm boot behavior does not energize outputs unexpectedly
- [ ] Measure output terminals with a multimeter before attaching the Andon

Success criterion: all four outputs can be toggled individually and remain OFF during boot unless deliberately commanded.

## Phase 3 - Andon integration

Proposed channel allocation:

| Channel | Function |
| --- | --- |
| OUT1 | Red |
| OUT2 | Yellow |
| OUT3 | Green |
| OUT4 | Buzzer |

This allocation can change after the GPIO mapping is known.

- [ ] Connect brown Andon wire to +12 V
- [ ] Connect color/buzzer control wires to switched outputs
- [ ] Verify red
- [ ] Verify yellow
- [ ] Verify green
- [ ] Verify buzzer
- [ ] Test multiple outputs simultaneously
- [ ] Add fuse and strain relief for the final demo assembly

## Phase 4 - Home Assistant

- [ ] Enable encrypted ESPHome native API
- [ ] Add device to Home Assistant
- [ ] Create four low-level diagnostic output entities
- [ ] Add semantic Andon mode control
- [ ] Add Home Assistant dashboard card
- [ ] Verify device remains operational when Home Assistant is stopped

## Phase 5 - MQTT

- [ ] Connect ESPHome to Mosquitto
- [ ] Disable duplicate Home Assistant MQTT entity discovery when native API is used
- [ ] Configure MQTT birth and last-will status
- [ ] Implement custom command topics
- [ ] Implement retained state topics
- [ ] Verify using MQTT Explorer
- [ ] Verify using Node-RED

## Phase 6 - UNS demonstrator

- [ ] Finalize topic hierarchy
- [ ] Add machine-state abstraction
- [ ] Add simulated production asset
- [ ] Add event messages with reason/source
- [ ] Add alarm acknowledge input
- [ ] Demonstrate one-to-many data consumption
- [ ] Demonstrate operation without Home Assistant

Example scenario:

1. Machine simulator reports RUNNING.
2. Andon shows green.
3. Machine temperature crosses warning threshold.
4. A rule publishes WARNING.
5. Andon shows yellow.
6. Temperature crosses fault threshold.
7. A rule publishes FAULT.
8. Andon shows red and buzzer.
9. Operator acknowledges alarm.
10. Buzzer stops while red remains active.
11. Machine recovers.
12. Andon returns to green.

## Phase 7 - Portable lab kit

- [ ] Decide between customer/lab Wi-Fi and dedicated demo Wi-Fi
- [ ] Build portable MQTT broker
- [ ] Add compact travel router if needed
- [ ] Add Node-RED
- [ ] Add MQTT Explorer or browser dashboard
- [ ] Document network setup
- [ ] Create one-command demo startup procedure

Target: the demo can be unpacked and made operational without depending on a customer's Home Assistant installation.

## Later ideas

- Physical acknowledge/reset button
- Stack-light test button
- Ethernet-capable controller variant
- TLS MQTT
- Sparkplug B comparison
- OPC UA to MQTT bridge
- Simulated PLC
- Edge AI event generation
- Home Assistant and industrial dashboard side by side
