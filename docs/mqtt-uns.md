# MQTT and UNS model

Last reviewed: 2026-09-19

## Current position

MQTT is working today.

UNS-specific custom topics are not.

The current project baseline uses ESPHome's standard MQTT entity topics. This is sufficient for:

- connecting the physical Andon to Mosquitto
- reading state
- changing Andon mode
- muting/unmuting the buzzer
- acknowledging alarms
- testing publish and subscribe behavior
- moving the device to another broker without reflashing

A separate custom Hupla/UNS topic model remains a future experiment.

## Runtime broker provisioning

Firmware 0.5.0 exposes these fields through the Andon web interface:

- MQTT Broker
- MQTT Port
- MQTT Username
- MQTT Password
- MQTT Topic Prefix
- Save & Connect MQTT
- Disconnect MQTT
- MQTT Connected

Broker hostname/IP, port and credentials are applied at runtime.

Current home-lab broker:

```text
192.168.129.15:1883
```

This broker has been validated.

A second independent broker has not yet been tested.

## Current working MQTT topics

ESPHome currently creates the active topic hierarchy.

### Availability

```text
andon-light-01/status
```

Standard ESPHome availability payloads are:

```text
online
offline
```

### Andon mode

Command:

```text
andon-light-01/select/andon_mode/command
```

State:

```text
andon-light-01/select/andon_mode/state
```

Useful command payloads:

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

### Buzzer Mute Override

Command:

```text
andon-light-01/switch/buzzer_mute_override/command
```

State:

```text
andon-light-01/switch/buzzer_mute_override/state
```

Payloads:

```text
ON
OFF
TOGGLE
```

### Alarm acknowledge

Command:

```text
andon-light-01/button/acknowledge_alarm/command
```

Payload:

```text
PRESS
```

These paths have been validated using Home Assistant's MQTT listen/publish tools.

## Home Assistant role

Home Assistant currently has two distinct relationships with the device:

1. normal control through the encrypted ESPHome native API
2. MQTT test client through the MQTT integration

MQTT discovery is disabled in ESPHome to prevent duplicate Home Assistant entities.

Mosquitto itself does not require a manual "Andon device" object. The ESP32 and Home Assistant are simply MQTT clients connected to the broker.

## Topic Prefix field

The local web UI currently contains an MQTT Topic Prefix field with initial value:

```text
hupla/demo/factory01/line01/andon01
```

Important:

- it is saved as runtime configuration
- it is reserved for later custom semantic topics
- it does not currently alter the automatic ESPHome MQTT topics
- changing it today does not move the existing `andon-light-01/...` topics

This distinction must remain explicit in project documentation.

## Deferred UNS model

The original concept remains useful, but it is now treated as a future layer rather than current functionality.

Possible namespace:

```text
hupla/demo/factory01/line01/andon01
```

Possible future topics:

```text
.../status/online
.../command/mode
.../command/acknowledge
.../command/buzzer_muted
.../state/mode
.../state/acknowledged
.../state/buzzer_muted
.../state/outputs
.../event
```

Possible policy:

| Topic type | Retain |
| --- | --- |
| availability | yes |
| current state | yes |
| command | no |
| event | no |

## Why not implement it now

The current ESPHome MQTT interface already proves:

- broker connection
- pub/sub
- read/write state
- physical response
- broker portability

Adding a second custom topic API now would mainly add code and maintenance.

The UNS layer becomes valuable when the demo has a real upstream producer, for example:

- PLC
- SCADA
- MES
- Node-RED
- OPC UA gateway
- industrial data platform

At that point the design question should be whether the Andon consumes a dedicated command such as `FAULT`, or subscribes to shared machine/process state.

## Future semantic example

A future UNS might publish machine state once:

```text
factory/site01/line01/machine01/state
```

Example payload:

```json
{
  "state": "FAULT",
  "reason": "temperature_high",
  "temperature": 92.4
}
```

Consumers could include:

```text
MES
dashboard
historian
AI/analytics
Andon
```

The Andon would then become a physical consumer of operational context rather than a device that must receive a vendor-specific lamp command.

## Future extensions

Only when a concrete demo requires them:

- custom semantic MQTT API
- retained state
- custom birth and last will
- event messages
- machine-state abstraction
- Sparkplug B comparison
- OPC UA to MQTT bridge
- MQTT TLS
- client certificate provisioning
- ACL examples
