# First bring-up procedure

This procedure deliberately stops before connecting the 12 V Andon.

## Goal

Reach this state first:

```text
PC -> USB-UART -> ESP32 -> ESPHome -> Wi-Fi -> OTA -> fallback Wi-Fi recovery
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

## Step 7 - Verify ESPHome OTA

Make a harmless change to the device configuration and upload over Wi-Fi from ESPHome.

Once this works, normal development no longer requires the UART adapter.

## Step 8 - Verify fallback AP and captive portal

1. Temporarily make the configured Wi-Fi networks unavailable.
2. Wait for the ESP32 fallback access point to appear.
3. Connect a phone or laptop to the fallback SSID.
4. Open the captive portal.
5. Confirm that a new Wi-Fi network can be selected/provisioned.
6. Restore the normal test network and verify the ESP32 reconnects.

Do not use a production or sensitive Wi-Fi password for this first test.

## Step 9 - Verify web OTA

1. Open the authenticated ESPHome web interface.
2. Locate the OTA update section.
3. Use a harmless precompiled firmware build.
4. Upload it through the browser.
5. Confirm the device reboots and reconnects.

UART remains the recovery method if either OTA path fails.

## Step 10 - Map MOSFET outputs

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

## Step 11 - 12 V integration

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
