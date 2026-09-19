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

Status: core bring-up complete

- [x] Solder the six-pin UART programming header
- [x] Detect Silicon Labs CP210x adapter on Windows
- [x] Connect TXD -> RX, RXD -> TX, GND -> GND
- [x] Power the ESP32 board independently during serial flashing
- [x] Enter bootloader mode using IO0
- [x] Complete first ESPHome flash
- [x] Join home Wi-Fi
- [x] Confirm encrypted ESPHome native API
- [x] Verify ESPHome OTA update using a friendly-name change
- [ ] Verify browser-based web OTA with a firmware.ota.bin image
- [ ] Verify UART as a deliberate recovery path

Observed runtime during successful OTA test:

- ESPHome 2026.8.2
- ESP32 rev 3.1, dual core
- native API handshake successful
- OTA service available
- Web Server OTA component loaded

Success criterion achieved for normal development: the ESP32 can now be updated wirelessly without the UART adapter.

## Phase 1B - Standalone fallback interface

Status: web portal tested successfully

Design decision: use the ESPHome fallback AP and normal local web server together, rather than relying on the captive portal as the primary fallback UI.

Target behavior:

```text
Known Wi-Fi available
  -> join normal network
  -> Home Assistant / local web / later MQTT

No known Wi-Fi available
  -> start Andon-Setup
  -> browse to 192.168.4.1
  -> control Red / Yellow / Green / Buzzer
  -> enter new SSID + password
  -> Connect & Save WiFi
```

- [x] Remove `captive_portal:` from the firmware baseline
- [x] Enable protected fallback AP `Andon-Setup`
- [x] Enable authenticated Web Server v3 with local assets
- [x] Add Red / Yellow / Green / Buzzer controls to the local web UI
- [x] Add SSID and password fields to the local web UI
- [x] Add `wifi.configure` action with persistent save
- [ ] Verify direct control while only connected to the fallback AP
- [ ] Verify saved Wi-Fi survives reboot

## Phase 2 - MOSFET output mapping

Expected board-family mapping, still to be verified:

| Output | Expected GPIO |
| --- | ---: |
| OUT1 | GPIO16 |
| OUT2 | GPIO17 |
| OUT3 | GPIO26 |
| OUT4 | GPIO27 |

- [ ] Verify GPIO16 -> OUT1
- [ ] Verify GPIO17 -> OUT2
- [ ] Verify GPIO26 -> OUT3
- [ ] Verify GPIO27 -> OUT4
- [ ] Confirm whether GPIO high means MOSFET ON
- [ ] Confirm boot behavior does not energize outputs unexpectedly
- [ ] Measure output terminals with a multimeter before attaching the Andon

Success criterion: all four outputs can be toggled individually and remain OFF during boot unless deliberately commanded.

## Phase 2B - Alarm logic and annunciation

- [x] Define semantic color convention
- [x] Define 0.5 Hz, 1 Hz and 2 Hz visual flash patterns
- [x] Define audible patterns for warning/fault severities
- [x] Define Acknowledge behavior
- [x] Add global Buzzer Mute Override design
- [x] Expose mute override to local web UI and Home Assistant
- [ ] Deploy firmware 0.4.0
- [ ] Verify visual flash timing
- [ ] Verify all buzzer patterns
- [ ] Verify Acknowledge on WARNING / FAULT / CRITICAL / EMERGENCY
- [ ] Verify mute override survives reboot
- [ ] Verify mute override suppresses Manual Buzzer

## Phase 3 - Andon integration

Proposed channel allocation:

| Channel | Function |
| --- | --- |
| OUT1 | Red |
| OUT2 | Yellow |
| OUT3 | Green |
| OUT4 | Buzzer |

- [x] Verify 12 V DC input powers both the controller and ESP32 in the real setup
- [x] Connect brown Andon wire to +12 V
- [x] Connect color/buzzer control wires to switched outputs
- [x] Verify red
- [x] Verify yellow
- [x] Verify green
- [x] Verify buzzer
- [x] Test multiple outputs and identify TD-50 internal red/yellow interaction
- [x] Confirm the red/yellow problem follows the Andon when MOSFET channels are swapped
- [x] Decide semantic modes will use one color at a time
- [ ] Add fuse and strain relief for the final demo assembly

## Phase 4 - Home Assistant

- [x] Encrypted ESPHome native API running
- [x] Device visible and online in ESPHome Device Builder
- [ ] Add four low-level diagnostic output entities after GPIO verification
- [x] Define semantic Andon mode control in firmware baseline 0.4.0
- [ ] Deploy and verify semantic Andon mode control
- [ ] Add Home Assistant dashboard card
- [ ] Verify Buzzer Mute Override in Home Assistant
- [ ] Verify device remains operational when Home Assistant is stopped

## Phase 5 - MQTT

Home broker baseline:

- [x] Install Mosquitto Broker in Home Assistant
- [x] Create dedicated Andon MQTT credentials
- [x] Identify Home Assistant LAN broker address: 192.168.129.15
- [x] Add MQTT secrets to ESPHome
- [x] Design runtime broker configuration
- [x] Add firmware 0.5.0 web fields for broker / port / username / password / topic prefix
- [x] Add Save & Connect MQTT action
- [x] Add MQTT Connected status
- [ ] Deploy firmware 0.5.0
- [ ] Verify connection to home Mosquitto
- [ ] Change to a second broker from the web UI without reflashing
- [ ] Verify broker settings survive reboot
- [ ] Disable duplicate Home Assistant MQTT entity discovery when native API is used
- [ ] Configure semantic MQTT birth and last-will status
- [ ] Implement custom command topics
- [ ] Implement retained state topics
- [ ] Verify using MQTT Explorer
- [ ] Verify using Node-RED

Portability target:

```text
Connect Andon to site Wi-Fi
  -> open local Andon web UI
  -> enter MQTT broker hostname/IP
  -> enter port / username / password
  -> Save & Connect MQTT
  -> no ESPHome rebuild or reflash required
```

Initial runtime provisioning supports ordinary MQTT TCP with username/password. Runtime provisioning of customer CA certificates or mutual-TLS client certificates remains a later advanced feature.

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
