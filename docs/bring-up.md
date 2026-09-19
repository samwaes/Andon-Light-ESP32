# Bring-up and commissioning status

## Current state

Core bring-up is complete.

Confirmed on 2026-09-19:

- UART header soldered
- Silicon Labs CP210x USB-to-UART adapter detected by Windows
- first ESPHome flash completed
- ESP32 online over Wi-Fi
- encrypted ESPHome native API working
- OTA update working
- OTA test performed by changing the friendly name
- ESPHome 2026.8.2
- ESP32 rev 3.1, dual core

The 12 V Andon has not yet been connected.

## Serial recovery wiring

```text
USB-UART       ESP32 board
TXD         -> RX
RXD         -> TX
GND         -> GND
```

Leave the UART adapter 5 V, 3.3 V and DTR pins unconnected when the ESP32 board is powered separately.

To enter the serial bootloader:

1. Remove ESP32 power.
2. Hold IO0 to GND.
3. Apply ESP32 power.
4. Flash firmware.
5. Remove power after the flash completes.
6. Remove IO0-to-GND.
7. Reapply power for normal boot.

UART is now considered the recovery method, not the normal update method.

## Normal update method

Use ESPHome OTA from Device Builder.

This has been verified successfully.

## Next test 1 - deploy standalone web fallback

Deploy the firmware baseline in:

```text
esphome/andon-light.yaml.example
```

Goals:

- authenticated local Web Server v3
- local assets
- fallback AP called `Andon-Setup`
- no captive portal intercepting the fallback UI
- direct output switches
- new SSID/password fields
- `Connect & Save WiFi` button

### Verify on normal Wi-Fi

1. Update wirelessly.
2. Confirm the device returns online.
3. Open `http://andon-light-01.local/`.
4. Confirm the local web interface loads.
5. Confirm Home Assistant API still connects.

### Verify fallback AP

1. Temporarily make all configured infrastructure Wi-Fi networks unavailable.
2. Wait for `Andon-Setup` to appear.
3. Connect a phone or laptop to it.
4. Browse manually to `http://192.168.4.1/`.
5. Confirm the same local Andon controls are available.
6. Enter a test SSID/password.
7. Press `Connect & Save WiFi`.
8. Verify the ESP joins the new network.
9. Reboot and verify the saved Wi-Fi remains available.

## Next test 2 - verify Web OTA

1. Build an OTA firmware image.
2. Open the authenticated local web interface.
3. Use the OTA Update section.
4. Upload the OTA firmware binary.
5. Confirm the ESP reboots and returns online.

Do not use a factory image for browser OTA.

## Next test 3 - map MOSFET outputs

Expected mapping:

| Output | Expected GPIO |
| --- | ---: |
| OUT1 | GPIO16 |
| OUT2 | GPIO17 |
| OUT3 | GPIO26 |
| OUT4 | GPIO27 |

Do not connect the Andon yet.

For each channel:

1. Configure the GPIO switch with `restore_mode: ALWAYS_OFF`.
2. Power the board.
3. Toggle the output from the local web UI or Home Assistant.
4. Measure the matching MOSFET terminal with a multimeter.
5. Confirm the channel turns off again.
6. Record the verified mapping.

## Next test 4 - 12 V power

The final design uses the board's DC input as the single power source.

Target:

```text
12 V PSU
   |
   +--> controller DC input
   |       |
   |       +--> onboard conversion --> ESP32
   |
   +--> Andon common +12 V
```

Before connecting the Andon:

1. Disconnect USB-C power.
2. Connect 12 V DC to the controller's marked DC input.
3. Verify polarity before applying power.
4. Apply 12 V.
5. Confirm the ESP32 boots and reconnects to Wi-Fi.
6. Confirm OTA/log access still works.

## Next test 5 - connect the Andon

Only after the GPIO and 12 V tests pass:

```text
Brown   -> +12 V common
Red     -> verified Red MOSFET output
Yellow  -> verified Yellow MOSFET output
Green   -> verified Green MOSFET output
Orange  -> verified Buzzer MOSFET output
```

Connect and test one function at a time. Test the buzzer last.

## Stop conditions

Do not continue if:

- the ESP32 becomes unusually hot
- an output activates unexpectedly during boot
- the 12 V input polarity is uncertain
- the MOSFET terminal behavior does not match the expected low-side switching
- the ESP32 fails to boot from the DC input alone
