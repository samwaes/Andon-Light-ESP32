# MQTT and UNS model

## Objective

MQTT is the portable interface of the Andon demonstrator.

The MQTT model should represent industrial meaning, not ESP32 implementation details.

A client should be able to request FAULT without knowing which GPIO controls the red lamp or buzzer.

## Namespace

Initial development namespace:

```text
hupla/demo/factory01/line01/andon01
```

This is intentionally simple and can later be aligned with a broader ISA-95, UNS or customer naming model.

## Proposed topics

### Availability

```text
hupla/demo/factory01/line01/andon01/status/online
```

Payload:

```text
online
offline
```

Retain: yes.

Use MQTT birth and last-will behavior.

### Command mode

```text
hupla/demo/factory01/line01/andon01/command/mode
```

Accepted payloads:

```text
OFF
RUNNING
WARNING
FAULT
STOPPED
MAINTENANCE
```

Retain: no.

### Raw output command

For commissioning only:

```text
hupla/demo/factory01/line01/andon01/command/outputs
```

Example:

```json
{
  "red": false,
  "yellow": true,
  "green": false,
  "buzzer": false
}
```

Retain: no.

### Current mode

```text
hupla/demo/factory01/line01/andon01/state/mode
```

Example:

```text
WARNING
```

Retain: yes.

### Current outputs

```text
hupla/demo/factory01/line01/andon01/state/outputs
```

Example:

```json
{
  "red": false,
  "yellow": true,
  "green": false,
  "buzzer": false
}
```

Retain: yes.

### Events

```text
hupla/demo/factory01/line01/andon01/event
```

Example:

```json
{
  "event": "alarm_acknowledged",
  "source": "operator",
  "mode": "FAULT"
}
```

Retain: no.

## Machine simulator

The UNS demo should not stop at direct Andon control.

A simulated asset can publish:

```text
hupla/demo/factory01/line01/machine01/state
```

Example:

```json
{
  "state": "RUNNING",
  "speed": 118,
  "target_speed": 120,
  "temperature": 54.2,
  "quality": 98.7
}
```

A rule engine or Node-RED flow can derive the Andon condition.

Example:

```text
temperature < 75     -> RUNNING
temperature >= 75    -> WARNING
temperature >= 90    -> FAULT
```

The Andon then becomes one subscriber to the same information used by dashboards, analytics and other consumers.

## UNS demonstration value

The demo should show:

1. Producer and consumer decoupling
2. One data publication feeding multiple consumers
3. Semantic information instead of hardware commands
4. Retained current state
5. Non-retained events
6. Device availability
7. Edge behavior without Home Assistant

## Later extensions

- richer asset hierarchy
- alarm IDs
- timestamps sourced from an NTP-synchronized ESP32 or upstream system
- acknowledge/reset workflow
- Sparkplug B comparison
- OPC UA bridge
- MQTT TLS
- per-device authentication and ACLs
