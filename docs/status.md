# Project status - 2026-09-19

## Working now

- ESP32-WROOM-32E controller operational
- UART header soldered and serial recovery path proven
- ESPHome 2026.8.2 running
- Wi-Fi and encrypted native API working
- ESPHome OTA working
- local ESPHome web portal tested successfully
- fallback/local web architecture implemented
- controller and Andon running from the 12 V setup
- red, yellow, green and buzzer individually controllable
- TD-50 internal red/yellow interaction isolated by swapping output channels

## Confirmed Andon hardware limitation

The HNTD TD-50 does not reliably support red + yellow simultaneously.

The problem follows the Andon function when the wires are moved to other MOSFET outputs, which rules out one specific MOSFET channel as the cause.

Firmware design consequence:

- semantic modes use one color at a time
- manual multi-color control remains diagnostic only

## Firmware 0.4.0 target

New semantic controls:

- Andon Mode
- Buzzer Mute Override
- Acknowledge Alarm
- Clear / OFF
- Alarm Acknowledged state
- Manual Red / Yellow / Green / Buzzer

Mode set:

```text
OFF
READY
RUNNING
STARTING
ATTENTION
WARNING
URGENT_WARNING
FAULT
CRITICAL
EMERGENCY
STOPPED
MAINTENANCE
MANUAL
```

Flash patterns:

- slow: 0.5 Hz
- normal: 1 Hz
- fast: 2 Hz

Acknowledge:

- stops audible indication
- converts active warning/fault flashing to steady
- does not clear the mode

Buzzer Mute Override:

- forcibly disables the buzzer
- leaves visual mode unchanged
- leaves acknowledge state unchanged
- is available in both local web UI and Home Assistant
- is restored across reboot for demo convenience

## Next tests

1. Deploy firmware 0.4.0.
2. Verify mode selector appears in both web UI and Home Assistant.
3. Verify mute override appears and suppresses every buzzer pattern.
4. Verify mute survives reboot.
5. Verify WARNING, FAULT, CRITICAL and EMERGENCY timing.
6. Verify Acknowledge transitions flashing alarm to steady visual.
7. Add MQTT/UNS after local behavior is stable.
