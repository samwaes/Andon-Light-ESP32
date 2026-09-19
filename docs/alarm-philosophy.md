# Andon alarm philosophy

Last reviewed: 2026-09-19

## Purpose

The Andon communicates operational meaning rather than raw GPIO state.

The design follows common industrial signal conventions:

- green = normal / ready / running
- yellow = abnormal / attention / intervention may be required
- red = fault / stop / critical condition
- audible indication = draw attention to a new condition
- acknowledge = silence the audible annunciation while retaining the visible condition
- clear/reset = separate from acknowledge

There is no universal exact flash frequency for every factory. This demonstrator uses a small consistent set that is easy to understand and test.

Reference concepts used when the model was defined:

- IEC 60204-1 indicator-light color conventions
- ISA-18.1 annunciator sequence concepts
- ISA-18.2 alarm management concepts
- common industrial signal-tower flash patterns

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

This mode set is deployed in firmware 0.5.0.

## Acknowledge behavior

For WARNING, URGENT_WARNING, FAULT, CRITICAL and EMERGENCY:

1. the new condition flashes and may sound the buzzer
2. operator acknowledges
3. buzzer stops
4. active color becomes steady
5. mode remains active until cleared or replaced

Acknowledge therefore does not mean that the underlying process condition has been resolved.

Current acknowledge paths:

- local web interface
- Home Assistant native ESPHome API
- standard MQTT button command

MQTT command:

```text
andon-light-01/button/acknowledge_alarm/command
payload: PRESS
```

## Buzzer Mute Override

Buzzer Mute Override is deliberately separate from acknowledgement.

When enabled:

- visual mode continues
- severity/mode remains unchanged
- acknowledge state remains unchanged
- physical buzzer is forced off
- manual buzzer activation is suppressed by firmware logic

Current control paths:

- local web interface
- Home Assistant native ESPHome API
- standard MQTT switch command

MQTT example:

```text
andon-light-01/switch/buzzer_mute_override/command
payload: ON
```

The firmware uses a restoring switch mode so the mute setting is intended to survive reboot. A deliberate full power-cycle validation remains open.

## Hardware-specific constraint

The physical HNTD TD-50 has a confirmed internal red/yellow interaction.

Observed:

- red + green works
- yellow + green works
- red + yellow is unreliable
- swapping MOSFET channels does not move the problem

All semantic modes therefore use one lamp color at a time.

MANUAL remains available for diagnostics, but multi-color combinations are not considered valid operational states for this unit.

## Current MQTT representation

Today the active MQTT representation is the standard ESPHome entity/state model.

Andon mode:

```text
andon-light-01/select/andon_mode/command
andon-light-01/select/andon_mode/state
```

Buzzer mute:

```text
andon-light-01/switch/buzzer_mute_override/command
andon-light-01/switch/buzzer_mute_override/state
```

Acknowledge:

```text
andon-light-01/button/acknowledge_alarm/command
```

## Future semantic UNS model

A future UNS implementation may represent the process condition, operator acknowledgement and local presentation override independently.

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

This custom representation is not implemented in firmware 0.5.0. It remains a later UNS experiment.
