# First bring-up procedure

This procedure deliberately stops before connecting the 12 V Andon.

## Goal

Reach this state first:

```text
PC -> USB-UART -> ESP32 -> ESPHome -> Wi-Fi -> OTA
```

Only after this works should the MOSFET channels and Andon lamp be connected.

## Step 1 - Solder UART header

Solder a six-pin header into:

```text
5V | TX | RX | GND | GND | IO0
```

Inspect for solder bridges before applying power.

## Step 2 - Connect UART

```text
USB-UART       ESP32 board
TXD         -> RX
RXD         -> TX
GND         -> GND
```

Leave these unconnected:

```text
USB-UART +5V
USB-UART 3V3
USB-UART DTR
```

Power the ESP32 board independently.

## Step 3 - Enter bootloader

1. Remove ESP32 power.
2. Hold IO0 to GND.
3. Apply ESP32 power.
4. Start flashing.

If the physical IO0 button proves reliable, it can be used instead of a temporary jumper.

## Step 4 - Flash minimal ESPHome firmware

Use the configuration in:

```text
esphome/andon-light.yaml.example
```

Initially, do not configure the MOSFET GPIOs.

## Step 5 - Verify boot

Expected checks:

- serial output is readable
- ESP32 identifies itself normally
- no repeated resets
- Wi-Fi connects
- IP address is assigned

## Step 6 - Verify Home Assistant API

If testing at home:

- add the ESPHome device to Home Assistant
- confirm it remains online
- confirm encrypted native API connectivity

## Step 7 - Verify OTA

Make a harmless change to the device configuration and upload over Wi-Fi.

Once OTA works, the UART adapter is only needed for recovery.

## Step 8 - Map MOSFET outputs

Do not connect the Andon yet.

Determine which GPIO drives each MOSFET channel.

For each candidate channel:

1. Configure only one GPIO.
2. Boot with output OFF.
3. Toggle it manually.
4. Measure OUT+/OUT- with a multimeter or use a safe test load.
5. Record the result.
6. Repeat for all four channels.

Update `docs/hardware.md` and the ESPHome configuration with confirmed mappings.

## Step 9 - 12 V integration

After output behavior is understood:

1. Power down.
2. Connect common 0 V between controller and supply.
3. Connect Andon brown to +12 V.
4. Connect one light channel only.
5. Test.
6. Add remaining channels one at a time.
7. Test the buzzer last.

## Stop conditions

Do not continue if:

- the ESP32 becomes unusually hot
- the UART adapter is configured for 5 V logic
- a MOSFET output is active unexpectedly during boot
- output terminal polarity cannot be established
- there is uncertainty about the 12 V supply polarity
