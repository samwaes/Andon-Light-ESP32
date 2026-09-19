# Hardware

## 1. Andon light

Model: HNTD TD-50

Configuration observed from the product label:

- 12 V DC
- constant-light variant
- red, yellow and green indication
- buzzer output

### Cable mapping

| Cable color | Function |
| --- | --- |
| Brown | Common positive, +12 V |
| Red | Red lamp control |
| Yellow | Yellow lamp control |
| Green | Green lamp control |
| Orange | Buzzer control |

The wiring diagram on the unit shows a common positive supply. Each function is activated by switching its individual control wire toward the negative/0 V side.

## 2. ESP32 quad MOSFET board

Received hardware is marked as an ESP32 MOS x4 board and contains an ESP32-WROOM-32E module.

Observed features:

- ESP32-WROOM-32E
- four MOSFET channels
- channel terminal markings OUT1 to OUT4
- wide-voltage DC input
- USB-C power connector
- IO0 pushbutton
- UART programming header
- exposed ESP32 GPIO footprint

MOSFET marking observed on the board: NCE6020AK.

### UART programming header

The board silkscreen identifies:

```text
5V | TX | RX | GND | GND | IO0
```

The six-pin UART header has now been soldered and was used for the successful first flash.

### GPIO breakout

The back of the board exposes standard ESP32 signals including GPIOs, power and ground. These are useful for later additions such as an acknowledge button.

### GPIO mapping to verify

The expected mapping for this ESP32 MOS x4 board family is:

| MOSFET output | Expected ESP32 GPIO |
| --- | ---: |
| OUT1 | GPIO16 |
| OUT2 | GPIO17 |
| OUT3 | GPIO26 |
| OUT4 | GPIO27 |

Do not treat this mapping as confirmed until it is measured on the received board. Remaining checks:

- verify GPIO16 -> OUT1
- verify GPIO17 -> OUT2
- verify GPIO26 -> OUT3
- verify GPIO27 -> OUT4
- active-high versus active-low GPIO behavior
- exact relationship between each OUT+ and OUT- terminal
- output state during ESP32 reset/boot

## 3. USB-to-UART adapter

The received Silicon Labs CP210x adapter exposes:

```text
3V3 | GND | +5V | TXD | RXD | DTR
```

For initial flashing:

```text
USB-UART       ESP32
TXD         -> RX
RXD         -> TX
GND         -> GND
```

Do not connect +5 V or 3.3 V from the UART adapter when the ESP32 board is independently powered.

The adapter was detected successfully by Windows as a Silicon Labs CP210x USB-to-UART Bridge on COM7 during bring-up. The UART logic level must remain 3.3 V compatible with the ESP32.

## 4. Initial bootloader procedure

1. Turn ESP32 board power off.
2. Connect USB-UART TXD, RXD and GND.
3. Hold IO0 to GND.
4. Apply power to the ESP32 board.
5. Start firmware flashing.
6. After flashing, remove power.
7. Release/remove IO0-to-GND.
8. Reapply power for normal boot.

The physical IO0 pushbutton may be used instead of a jumper once its behavior is confirmed.

## 5. Proposed Andon connection

Expected low-side topology:

```text
12 V PSU +  ------------------ Brown Andon wire
     |
     +------------------------ ESP32 board VIN+

12 V PSU 0 V ---------------- ESP32 board GND

ESP32 OUT1 switched low ------ Red
ESP32 OUT2 switched low ------ Yellow
ESP32 OUT3 switched low ------ Green
ESP32 OUT4 switched low ------ Orange / buzzer
```

This topology has now been used with the real Andon and controller.

## 6. Power

The lamp is a 12 V device and the controller accepts 12 V input. The assembled setup has now been powered from the board's 12 V DC input, with the same supply powering the ESP32 electronics and the Andon load.

Before selecting the final supply, measure or obtain:

- current with green on
- current with yellow on
- current with red on
- current with buzzer on
- worst-case current with multiple functions active

Use a fused supply and appropriate wire/strain relief in the final enclosure.

## 7. Planned additions

Possible future I/O:

- acknowledge button
- reset button
- local mode selector
- maintenance/test button
- external sensor or simulated machine input


## 8. Confirmed TD-50 behavior

The real HNTD TD-50 has now been tested with multiple lamp combinations.

Observed:

- red + green can operate together
- yellow + green can operate together
- red + yellow do not operate correctly together
- with red + yellow requested, yellow becomes off or only very faint
- the behavior remains when the red/yellow wires are moved to different MOSFET channels

This channel-swap test localizes the interaction to the Andon itself rather than to one MOSFET channel, GPIO or ESPHome output.

Operational decision:

- semantic Andon modes use one color at a time
- multi-color output remains diagnostic/manual only
- the firmware does not rely on red + yellow simultaneous indication
