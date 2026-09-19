# Bring-up and commissioning record

Last reviewed: 2026-09-19

## Current state

Core bring-up and functional commissioning are complete.

Confirmed:

- UART header soldered
- Silicon Labs CP210x USB-to-UART adapter used successfully
- first ESPHome flash completed
- ESPHome 2026.8.2 running
- ESP32 rev 3.1 detected
- Wi-Fi working
- encrypted ESPHome native API working
- ESPHome OTA working
- local ESPHome web interface working
- fallback AP architecture deployed
- controller and Andon powered from 12 V
- all four MOSFET channels mapped
- red, yellow, green and buzzer tested
- semantic alarm modes tested
- Buzzer Mute Override and acknowledge tested
- firmware 0.5.0 deployed
- Mosquitto connection working
- MQTT listen and publish tested

The project is no longer in initial electrical bring-up. Remaining work is validation and physical finishing.

## Confirmed output mapping

| Output | GPIO | Function |
| --- | ---: | --- |
| OUT1 | GPIO16 | Red |
| OUT2 | GPIO17 | Yellow |
| OUT3 | GPIO26 | Green |
| OUT4 | GPIO27 | Buzzer |

The TD-50 uses brown as common +12 V.

```text
Brown   -> +12 V common
Red     -> OUT1 switched low
Yellow  -> OUT2 switched low
Green   -> OUT3 switched low
Orange  -> OUT4 switched low
```

## Power topology

Current working arrangement:

```text
12 V PSU
   |
   +--> ESP32 MOSFET board DC input
   |       |
   |       +--> onboard conversion --> ESP32
   |
   +--> Andon common +12 V
```

The four Andon function wires are switched toward 0 V by the MOSFET channels.

## Known hardware behavior

The real TD-50 does not reliably support red and yellow simultaneously.

Testing showed:

- red + green works
- yellow + green works
- red + yellow fails or leaves yellow only faintly lit
- swapping red/yellow to different MOSFET channels does not change the behavior

The limitation is therefore treated as internal to the Andon.

Semantic modes use one color at a time.

## Serial recovery wiring

```text
USB-UART       ESP32 board
TXD         -> RX
RXD         -> TX
GND         -> GND
```

Do not connect the UART adapter's 5 V or 3.3 V supply pins when the controller is powered separately.

### Bootloader procedure

1. Remove controller power.
2. Hold IO0 to GND.
3. Apply controller power.
4. Flash firmware.
5. Remove power.
6. Release IO0.
7. Reapply power for normal boot.

UART remains the recovery path, not the normal update path.

## Normal firmware update

Use ESPHome OTA from Device Builder.

This has been verified and is the preferred development workflow.

Live configuration normally resides at:

```text
/config/esphome/andon-light-01.yaml
```

## Local commissioning

Normal-network access:

```text
http://andon-light-01.local/
```

Fallback access point:

```text
SSID: Andon-Setup
Web:  http://192.168.4.1/
```

The web interface includes:

- semantic mode
- mute and acknowledge
- manual outputs
- Wi-Fi setup
- MQTT setup
- status
- restart

## MQTT commissioning

Current home-lab reference broker:

```text
192.168.129.15:1883
```

Verified:

- broker connection
- MQTT listening in Home Assistant
- MQTT publishing in Home Assistant
- mode commands
- mode state
- buzzer mute
- acknowledge

The broker can be configured at runtime from the Andon web interface.

## Remaining commissioning tests

### 1. Full power-cycle persistence

Verify after removing 12 V power completely:

- device returns normally
- outputs do not energize unexpectedly
- Wi-Fi configuration remains valid
- MQTT broker configuration remains valid
- expected mute state is restored

### 2. Second broker

Change broker address, port and credentials through the web UI only.

Verify:

- no firmware rebuild
- new broker connection succeeds
- standard ESPHome MQTT topics still work

### 3. Home Assistant independence

Stop or disconnect Home Assistant deliberately.

Verify:

- local web interface remains usable
- semantic modes still work
- MQTT remains functional if the broker is still available

### 4. Web OTA

Upload a valid OTA firmware image from the web interface and verify recovery.

Do not use a factory image for browser OTA.

### 5. Physical finish

Add and document:

- fuse
- strain relief
- enclosure
- cable labeling
- final assembly photographs

## Stop conditions for future hardware changes

Stop testing if:

- the ESP32 or board becomes unusually hot
- an output activates unexpectedly at boot
- 12 V polarity is uncertain
- wiring no longer matches the verified low-side topology
- a new load exceeds the board or power-supply rating
