# Andon alarm philosophy

## Purpose

The Andon should communicate operational meaning, not raw GPIO states.

The design follows common industrial stack-light conventions:

- green = normal / ready / running
- yellow = abnormal / attention / intervention may be required
- red = fault / stop / critical condition
- audible indication = draws operator attention to a new condition
- acknowledge = silence the audible annunciation while retaining a visible indication
- reset/clear = a separate action from acknowledge

There is no universal rule that fixes one exact flash frequency for every factory. This project therefore uses a small, consistent set of patterns that are easy to explain in a demo and align with common industrial signal-tower practice.

References:
- IEC 60204-1 indicator-light color conventions
- ISA-18.1 annunciator sequence concepts
- ISA-18.2 alarm management concepts
- Patlite industrial signal towers with 30/60/120 flashes per minute patterns

## Flash rates

| Pattern | Frequency | Timing |
| --- | ---: | --- |
| Slow | 0.5 Hz | 1 s on / 1 s off |
| Normal | 1 Hz | 0.5 s on / 0.5 s off |
| Fast | 2 Hz | 0.25 s on / 0.25 s off |

## Modes

| Mode | Visual | Audible before ACK | Meaning |
| --- | --- | --- | --- |
| OFF | off | off | inactive |
| READY | green steady | off | ready |
| RUNNING | green steady | off | normal operation |
| STARTING | green slow flash | off | startup / transition |
| ATTENTION | yellow steady | off | information / attention |
| WARNING | yellow slow flash | short beep every 2 s | abnormal, monitor/intervene |
| URGENT_WARNING | yellow 1 Hz | short beep every 1 s | intervention increasingly urgent |
| FAULT | red 1 Hz | 0.5 s on / 0.5 s off | fault / stopped by fault |
| CRITICAL | red 2 Hz | 0.25 s on / 0.25 s off | immediate intervention |
| EMERGENCY | red 2 Hz | continuous | highest demo severity |
| STOPPED | red steady | off | stopped, no new audible alarm |
| MAINTENANCE | yellow steady | off | maintenance/service state |
| MANUAL | raw outputs | manual | commissioning and diagnostics |

## Acknowledge behavior

For WARNING, URGENT_WARNING, FAULT, CRITICAL and EMERGENCY:

1. New condition flashes and may sound the buzzer.
2. Operator presses Acknowledge.
3. Buzzer stops.
4. The active color becomes steady.
5. The mode remains active until cleared or replaced.

Acknowledge therefore does not mean the underlying condition is resolved.

## Buzzer Mute Override

The project adds a separate global control:

```text
Buzzer Mute Override
```

This is deliberately different from alarm acknowledgement.

When enabled:

- every visual mode continues normally
- mode/severity remains unchanged
- acknowledge state remains unchanged
- the physical buzzer is forced OFF
- manual buzzer activation is also suppressed

The override is intended for demos, quiet lab work, presentations and commissioning where repeated audible alarms would be disruptive.

The override is exposed both in:

- ESPHome local web interface
- Home Assistant through the native ESPHome API

The mute state is restored after reboot so a device muted for a demo does not unexpectedly become noisy after a power cycle.

## Hardware-specific constraint

The current HNTD TD-50 has been physically tested and shows an internal interaction between red and yellow. Red + yellow cannot be used reliably at the same time even when the MOSFET outputs are swapped, which localizes the behavior to the Andon hardware.

Therefore all semantic modes use exactly one lamp color at a time.

The MANUAL mode remains available for diagnostics, but multi-color combinations are not considered valid operational states for this unit.

## Future MQTT / UNS model

The mute override should eventually be represented separately from the process alarm state.

Example:

```json
{
  "mode": "FAULT",
  "severity": "high",
  "acknowledged": false,
  "buzzer_muted": true,
  "visual": {
    "color": "red",
    "pattern": "flash",
    "frequency_hz": 1
  },
  "audible": {
    "requested_pattern": "normal",
    "effective": false
  }
}
```

This distinction is useful in a UNS because it separates:

- the machine/process condition
- operator acknowledgement
- local presentation overrides
