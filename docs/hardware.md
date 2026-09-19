# Hardware

Last reviewed: 2026-09-19

## 1. Andon light

Model: HNTD TD-50

Confirmed configuration:

- 12 V DC
- constant-light version
- red, yellow and green
- buzzer
- common positive supply

### Cable mapping

| Cable color | Function |
| --- | --- |
| Brown | Common +12 V |
| Red | Red lamp control |
| Yellow | Yellow lamp control |
| Green | Green lamp control |
| Orange | Buzzer control |

Each function is activated by switching its control wire toward 0 V.

## 2. ESP32 quad-MOSFET controller

Received controller:

- ESP32-WROOM-32E
- four MOSFET output channels
- NCE6020AK MOSFETs
- OUT1 to OUT4 screw-terminal outputs
- 5-60 V marked DC input range
- onboard conversion for ESP32 power
- USB-C power input
- IO0 pushbutton
- UART programming header
- exposed ESP32 GPIO footprint

The board has two DC input terminals plus four output pairs.

### Confirmed output mapping

| MOSFET output | ESP32 GPIO | Function |
| --- | ---: | --- |
| OUT1 | GPIO16 | Red |
| OUT2 | GPIO17 | Yellow |
| OUT3 | GPIO26 | Green |
| OUT4 | GPIO27 | Buzzer |

The mapping has been exercised on the real Andon.

## 3. UART programming header

Silkscreen:

```text
5V | TX | RX | GND | GND | IO0
```

The six-pin header is soldered and was used for the first successful flash.

## 4. USB-to-UART adapter

Silicon Labs CP210x adapter.

Adapter pins observed:

```text
3V3 | GND | +5V | TXD | RXD | DTR
```

Programming connection:

```text
USB-UART       ESP32 board
TXD         -> RX
RXD         -> TX
GND         -> GND
```

Do not connect +5 V or 3.3 V from the UART adapter when the controller is independently powered.

Windows detected the adapter as a Silicon Labs CP210x USB-to-UART Bridge during bring-up.

## 5. Working Andon connection

```text
12 V PSU +  ------------------ Brown Andon wire
     |
     +------------------------ controller DC input +

12 V PSU 0 V ---------------- controller GND

OUT1 switched low ----------- Red
OUT2 switched low ----------- Yellow
OUT3 switched low ----------- Green
OUT4 switched low ----------- Orange / buzzer
```

This topology is physically deployed and working.

## 6. Power

The current setup uses one 12 V source for:

- controller input
- onboard ESP32 power conversion
- Andon common +12 V

The final demo assembly should still include:

- correctly rated fused supply
- strain relief
- cable labeling
- protected enclosure

Exact current consumption has not been formally documented yet.

Useful future measurements:

- controller idle current
- each individual lamp current
- buzzer current
- worst-case supported combination

## 7. Confirmed TD-50 multi-color behavior

Observed on the physical unit:

- red + green works
- yellow + green works
- red + yellow does not operate correctly together
- with red + yellow requested, yellow turns off or becomes very faint
- the behavior follows the red/yellow functions when wires are moved to other MOSFET channels

This rules out one specific MOSFET channel, GPIO or software output as the cause.

Project decision:

- semantic modes use one color at a time
- MANUAL remains available for diagnostics
- no operational mode depends on red + yellow simultaneously

## 8. Boot and output safety

Firmware raw outputs use:

```yaml
restore_mode: ALWAYS_OFF
```

A deliberate full power-cycle test should still be documented to confirm that no visible output is energized unexpectedly during boot.

## 9. Possible future I/O

Potential additions:

- physical acknowledge button
- reset/test button
- local selector
- external sensor
- simulated machine input

These are optional and not required for the current MQTT demonstrator.
