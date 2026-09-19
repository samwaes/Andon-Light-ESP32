# Roadmap

Last reviewed: 2026-09-19

## Current project position

The core demonstrator is working.

Current baseline:

- firmware 0.5.0 deployed
- physical Andon connected
- semantic local alarm logic working
- local web commissioning working
- Home Assistant native API working
- Mosquitto connection working
- standard ESPHome MQTT read/write working
- runtime MQTT broker reconfiguration available through the web UI
- custom UNS topic model deferred

The roadmap now focuses on validation, portability and later industrial integration rather than basic bring-up.

## Milestone 1 - Hardware and ESP32 bring-up

Status: complete for normal use.

- [x] Select 12 V HNTD TD-50 Andon
- [x] Select ESP32 quad-MOSFET controller
- [x] Solder UART programming header
- [x] Flash ESPHome through CP210x UART
- [x] Confirm ESPHome 2026.8.2 operation
- [x] Confirm Wi-Fi
- [x] Confirm encrypted native API
- [x] Confirm ESPHome OTA
- [x] Power controller and Andon from 12 V
- [x] Verify GPIO16 -> OUT1 -> red
- [x] Verify GPIO17 -> OUT2 -> yellow
- [x] Verify GPIO26 -> OUT3 -> green
- [x] Verify GPIO27 -> OUT4 -> buzzer
- [x] Confirm the TD-50 red/yellow interaction is internal to the Andon
- [ ] Verify safe output behavior during a deliberate full power cycle
- [ ] Add final fuse and strain relief
- [ ] Photograph and archive the finished assembly
- [ ] Record final part/vendor references

## Milestone 2 - Local Andon behavior

Status: working.

- [x] Direct Red / Yellow / Green / Buzzer controls
- [x] Semantic mode model
- [x] 0.5 Hz, 1 Hz and 2 Hz flash patterns
- [x] Warning/fault buzzer patterns
- [x] Acknowledge behavior
- [x] Buzzer Mute Override
- [x] Manual / Diagnostics mode
- [x] Use one semantic color at a time because of TD-50 hardware behavior
- [ ] Explicitly verify mute-state persistence across a full power cycle
- [ ] Explicitly verify manual buzzer remains suppressed while mute override is enabled

## Milestone 3 - Local commissioning and updates

Status: normal web interface working.

- [x] Authenticated Web Server v3
- [x] Embedded local web assets
- [x] Protected fallback AP `Andon-Setup`
- [x] No captive portal dependency
- [x] Runtime Wi-Fi fields
- [x] `Connect & Save WiFi`
- [x] Restart control
- [x] ESPHome OTA
- [x] UART recovery path established
- [ ] Re-test direct control while connected only to `Andon-Setup`
- [ ] Verify runtime Wi-Fi credentials after a full power cycle
- [ ] Verify browser-based Web OTA with an OTA firmware image
- [ ] Perform one deliberate UART recovery test after the device is fully assembled

## Milestone 4 - Home Assistant

Status: working baseline.

- [x] Native ESPHome API
- [x] Device available in Home Assistant
- [x] Semantic Andon Mode
- [x] Buzzer Mute Override
- [x] Acknowledge
- [x] Raw diagnostic outputs
- [x] MQTT integration available for listen/publish testing
- [ ] Optional dedicated Home Assistant dashboard card
- [ ] Deliberately stop Home Assistant and verify the device continues operating independently

## Milestone 5 - MQTT portability

Status: working home-broker baseline.

- [x] Install Mosquitto Broker
- [x] Create dedicated Andon MQTT credentials
- [x] Validate broker at `192.168.129.15:1883`
- [x] Deploy firmware 0.5.0
- [x] Runtime broker hostname/IP field
- [x] Runtime port field
- [x] Runtime username/password fields
- [x] Save & Connect MQTT
- [x] Disconnect MQTT
- [x] MQTT Connected status
- [x] MQTT remains optional and does not force reboot when unavailable
- [x] Disable Home Assistant MQTT discovery to avoid duplicate entities
- [x] Verify MQTT listening from Home Assistant
- [x] Verify MQTT publishing from Home Assistant
- [x] Verify mode command/state through standard ESPHome MQTT topics
- [x] Verify Buzzer Mute Override over MQTT
- [x] Verify Acknowledge over MQTT
- [ ] Verify persisted broker settings after a full power cycle
- [ ] Connect to a second MQTT broker from the web UI without reflashing
- [ ] Test with an MQTT client outside Home Assistant, for example MQTT Explorer
- [ ] Verify MQTT operation while Home Assistant is offline

### Current decision

Keep the standard ESPHome MQTT interface as the current baseline.

Do not add a second custom MQTT API until a concrete industrial demo needs it.

The runtime Topic Prefix field remains reserved for that later step.

## Milestone 6 - Industrial semantic / UNS experiment

Status: deferred by design.

Possible future scope:

- [ ] Define the concrete producer, for example PLC, MES, SCADA, Node-RED or OPC UA gateway
- [ ] Define the machine/process state model
- [ ] Decide whether the Andon consumes a dedicated command topic or shared machine state
- [ ] Activate the runtime topic prefix
- [ ] Add custom semantic MQTT subscriptions
- [ ] Add retained state
- [ ] Add explicit birth/last-will under the custom namespace
- [ ] Add non-retained events
- [ ] Demonstrate one producer feeding multiple consumers
- [ ] Demonstrate Andon as a physical consumer of shared operational context
- [ ] Compare plain MQTT with Sparkplug B if useful
- [ ] Demonstrate OPC UA to MQTT/UNS bridge if useful

Potential namespace:

```text
hupla/demo/factory01/line01/andon01/
```

This milestone should start from a real demonstration question, not from protocol work alone.

## Milestone 7 - Portable demo kit

Status: optional later phase.

The Andon no longer requires a portable broker because it can be pointed at a site's existing broker at runtime.

A self-contained demo kit may still be useful where customer IT cannot provide Wi-Fi or MQTT.

Possible components:

- travel router
- local MQTT broker
- Node-RED
- MQTT Explorer or browser dashboard
- simulated PLC or machine
- OPC UA gateway
- one-command demo startup procedure

## Later ideas

- physical acknowledge/reset button
- local test button
- Ethernet-capable controller variant
- TLS broker support
- runtime CA/client-certificate provisioning
- Sparkplug B
- OPC UA bridge
- edge AI event generation
- MES-driven Andon scenario
- Home Assistant and industrial dashboard side by side
