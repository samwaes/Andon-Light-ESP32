# RFID reader addition

Status: planned hardware and firmware extension. No firmware changes have been implemented yet.

## Goal

Add the existing ELECHOUSE PN532 NFC/RFID Module V3 to the Andon ESP32 so a tag scan can be reported to Home Assistant and/or MQTT. Home Assistant or another external automation should decide what an authorized tag does, such as acknowledge an alarm or request a reset. This keeps the list of authorized tags out of the ESP32 firmware.

A tag UID is an identifier, not a strong security credential. Do not use UID-only checks to authorize a real safety function or access control that requires security.

## Hardware already available

- ESP32-WROOM-32E quad-MOSFET Andon controller
- ELECHOUSE PN532 NFC/RFID Module V3 supplied by the user
- The PN532 V3 has an integrated PCB antenna. The documented typical tag reading range is about 5–7 cm, depending on the tag and installation.
- The Andon controller exposes GPIO pads on its expansion footprint. GPIO16, GPIO17, GPIO26 and GPIO27 are already assigned to the four MOSFET outputs.

No separate RFID antenna is required for the first test. The V3 board does not show an external antenna connector in the supplied photos.

## First connection: short I²C wiring

For an initial bench test, place the PN532 close to the ESP32 and use the four-pin I²C header:

| PN532 V3 | Andon ESP32 controller |
| --- | --- |
| SDA | GPIO21 |
| SCL | GPIO22 |
| VCC | 3V3 supply |
| GND | GND |

Set the PN532 mode switch to I²C: switch 1 ON, switch 2 OFF. Confirm the switch orientation against the labels printed on the actual module before powering it.

Use short wires for the first test. Do not connect the PN532 to the MOSFET output terminals. The PN532 communicates with ESP32 GPIO pads and needs a common ground.

The module is specified for 3.3–5 V supply. Use 3.3 V for this ESP32 test and never apply 5 V to an ESP32 GPIO. Check the actual board pinout before soldering or applying power.

## Placement about two metres from the ESP32

Two metres refers to the reader-to-controller cable, not the tag reading distance. The tag still needs to be held within a few centimetres of the PN532 antenna.

Direct I²C wiring over two metres is not the recommended first installation. It may work on a quiet bench, but the bus is sensitive to cable capacitance and electrical noise. If the reader must be two metres away, consider:

1. Put a small ESP32 near the PN532, then send tag events to the main Andon controller or Home Assistant over Wi-Fi/MQTT.
2. Keep one ESP32 and add a matched differential I²C extender pair, for example boards based on PCA9615. This adds hardware at both ends and needs a cable and power plan.
3. Use HSU/UART with suitable differential line drivers for the cable, but this requires custom PN532 firmware support because the standard ESPHome PN532 component documents I²C and SPI, not HSU.

Do not assume that switching the PN532 to SPI automatically makes a two-metre cable reliable. SPI needs more signal conductors and is intended for a local connection.

## Firmware and Home Assistant/MQTT plan

- Configure the ESPHome I²C bus on GPIO21/GPIO22.
- Add the ESPHome PN532 I²C hub and a tag event handler.
- Emit the scanned UID as an event to Home Assistant and/or a dedicated MQTT topic.
- Keep authorization and action mapping outside the ESP32. For example, Home Assistant can map a tag to “acknowledge”, “request alarm reset” or “no action”.
- Avoid embedding a list of authorized UIDs in the firmware.
- Do not publish tag UIDs to an unauthenticated or public broker. Review MQTT access controls and avoid retaining scan events unless there is a clear need.

Before relying on this, check the precise ESPHome version and MQTT event configuration used by the project, then test known and unknown tags and Home Assistant/MQTT disconnect behavior.

## Suggested test sequence

1. Power down the Andon controller before wiring.
2. Solder or fit leads to the PN532 I²C header and the ESP32 expansion pads.
3. Set the PN532 switches to I²C.
4. Connect GND first, then VCC, SDA and SCL. Inspect for shorts and verify supply voltage before powering up.
5. Add I²C bus scanning and confirm that the PN532 appears.
6. Scan a tag and confirm that the UID event reaches Home Assistant or MQTT.
7. Test with Home Assistant and MQTT unavailable, then reconnect and verify the expected behavior.
8. Only after the short-wire test succeeds, evaluate the two-metre installation option.

## Relationship to physical buttons

Reset, acknowledge and sound mute controls are separate digital inputs and do not need to share I²C. Candidate GPIOs should be checked against the board footprint and final firmware pin allocation before wiring. A physical emergency-stop input can report an emergency state to the Andon, but it must not be treated as a machine safety circuit. A real emergency stop requires a suitable hardwired safety device and safety chain independent of the ESP32, Wi-Fi, Home Assistant and MQTT.

## Existing reference project: Adonno Tag Reader

The reader project you identified is [adonno/tagreader](https://github.com/adonno/tagreader). It is an ESPHome-based Home Assistant NFC reader built around a D1 Mini ESP8266 and a PN532. Its README also documents the I²C switch setting used by your PN532 V3.

Useful files and parts:

- [Project README and build instructions](https://github.com/adonno/tagreader/blob/master/README.md): hardware overview, PN532 I²C mode and Home Assistant tag use.
- [tagreader.yaml](https://github.com/adonno/tagreader/blob/master/tagreader.yaml): the ESPHome configuration. The relevant reference is its `i2c:`, `pn532_i2c:`, `on_tag:` and `homeassistant.tag_scanned` sections.
- [Schematics](https://github.com/adonno/tagreader/tree/master/Schematics) and [3D case files](https://github.com/adonno/tagreader/tree/master/STLs): useful if building a separate, standalone reader.

### How to reuse it for this Andon

Use Adonno's repository as a working reference for the PN532 wiring and the tag-reading flow. Adapt the PN532 I²C and tag-event portions into this project's existing ESP32 configuration. This Andon already has its own ESP32, Wi-Fi, Home Assistant API, MQTT client, local web interface and output logic.

Do not flash Adonno's `tagreader.yaml` unchanged onto the Andon controller. It targets an ESP8266 D1 Mini and defines a separate standalone reader. Its GPIO assignments and ESP8266-specific LED/buzzer outputs do not match the ESP32 quad-MOSFET Andon board. Copying its full configuration could conflict with the Andon outputs and existing services.

The useful adaptation pattern is:

1. Reuse the PN532 I²C mode and wiring concept.
2. Add the ESPHome PN532 hub and `on_tag` handling to the Andon YAML.
3. Send the scanned tag event to Home Assistant, where HA's tag registry and automations can map tags to acknowledge or reset actions.
4. If MQTT is also needed, publish the scan event through the Andon firmware's MQTT setup, with broker permissions that restrict access to the event topic.
5. Leave Adonno's optional WS2812 activity light, buzzer, tag-writing controls, captive portal and ESP8266 pin assignments out unless separately required and adapted.

The Adonno configuration calls `homeassistant.tag_scanned` for scanned UIDs, which is the useful model for keeping tag-to-action configuration in Home Assistant rather than embedding an allowlist in firmware. Its code is licensed under GPL-3.0. If code is copied into this repository, retain the required license and attribution notices and check GPL-3.0 obligations for the resulting distribution.

## References

- [ELECHOUSE PN532 V3 manual](https://www.elechouse.com/elechouse/images/product/PN532_module_V3/PN532_%20Manual_V3.pdf)
- [ESPHome PN532 component](https://esphome.io/components/binary_sensor/pn532/)
- [ESPHome I²C bus](https://esphome.io/components/i2c/)
