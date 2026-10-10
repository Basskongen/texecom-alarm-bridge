# Texecom Premier Elite

Bridges a Texecom Premier Elite panel to Home Assistant over MQTT with automatic
discovery. Tested against a Premier Elite 88 V5.06.00 with a ComIP on Com Port 1.

## How it works

The app opens one TCP connection to the ComIP/SmartCom and logs in twice on it:

- **Connect protocol** (what texecom2mqtt uses): live messages for zones, areas,
  outputs and users; arm, disarm, reset and omit.
- **Wintex/UDL protocol**: memory reads for voltages, tamper flags, the keypad
  display, the panel clock, PC Control and the event log. Writes are limited to a
  short whitelist in the code (PC Control byte, clock, banner text).

Only one client can be connected to the panel. Stop texecom2mqtt and close Wintex
before starting the app. After a dropped connection the panel refuses new
connections for about a minute; the app waits and retries automatically.

## Installation

1. Add this repository in Settings > Apps > App store > three dots > Repositories.
2. Install "Texecom Premier Elite".
3. Fill in `host` and `udl_code`. MQTT details are taken from the Mosquitto app
   automatically; set `mqtt_host` only if you run another broker.
4. Start the app and watch the log. The device "Texecom Premier Elite" appears
   under Settings > Devices & services > MQTT.

## Options

| Option | Default | Description |
|--------|---------|-------------|
| `host` | | IP address of the ComIP / SmartCom |
| `port` | 10001 | TCP port |
| `udl_code` | | UDL passcode (4 to 6 digits). Used for both logins |
| `mqtt_host`, `mqtt_port`, `mqtt_username`, `mqtt_password` | from Supervisor | Override the broker |
| `mqtt_prefix` | `texecom` | Topic prefix for state and command topics |
| `discovery_prefix` | `homeassistant` | MQTT discovery prefix |
| `max_zones` | 88 | Highest zone number to read at start-up |
| `max_areas` | 8 | Number of areas to read at start-up |
| `part_arm_1`, `part_arm_2`, `part_arm_3` | armed_home, armed_night, armed_vacation | Which Home Assistant arm state each part arm uses |
| `zone_config` | [] | Per-zone overrides: `number`, `name`, `device_class` |
| `area_config` | [] | Per-area overrides: `number`, `name`, `part_arm_1..3` |
| `legacy_prefix` | | MQTT prefix of the bridge this app replaces (see Migrating) |
| `poll_interval` | 5 | Seconds between memory polls |
| `read_only` | false | Refuse every command that changes the panel |
| `allow_keypress` | false | Accept virtual key presses on `texecom/cmd/keypress` |
| `log_level` | info | `debug` logs every frame (the UDL code is masked) |

Example overrides:

```yaml
zone_config:
  - number: 5
    name: Front door
    device_class: door
  - number: 74
    name: Porch motion
    device_class: motion
area_config:
  - number: 1
    name: House
    part_arm_1: armed_night
    part_arm_2: armed_home
```

## Migrating from another MQTT bridge

If Home Assistant already has entities from a previous Texecom-to-MQTT bridge, set
`legacy_prefix` to the MQTT prefix that bridge used (the one its topics start with).
The app then:

- publishes discovery with the same unique ids (`<prefix>.area.<slug>`,
  `<prefix>.zone.<slug>`) and the same device (identified by the panel serial number),
  so the existing entities, their names and your automations are kept;
- publishes `<prefix>/status`, `<prefix>/zone/<slug>`, `<prefix>/area/<slug>`,
  `<prefix>/log` and `<prefix>/power`, and listens on `<prefix>/area/<slug>/command`
  (`full_arm`, `part_arm_1..3`, `disarm`), `<prefix>/text` and `<prefix>/datetime`;
- puts the power values on the alarm panel's attributes, as before.

Use `zone_config` / `area_config` to repeat any names the old bridge was configured
with, so the slugs match. Stop the old bridge first; the panel accepts one client.
The last-will message only covers the app's own `status` topic, so if the app
crashes the legacy status topic may stay `online` until it restarts.

## Entities

- **Alarm control panel** per area that has zones. States: disarmed, arming,
  pending, armed_away (full), armed_home (part arm), triggered. Commands: arm
  away (full arm), arm home (part 1), arm night (part 2), arm vacation (part 3),
  disarm. A reset button per area.
- **Binary sensors** per used zone: open/active (device class from the zone type),
  tamper, alarm memory. A switch per zone for omit.
- **Outputs**: panel output 1-5 and digi output 1-8 (from Connect output messages).
- **PC Control 1-8** switches (names from the panel) and **Control Timer 1-8** sensors.
- **System**: panel and battery voltage (calibrated against Wintex over 11.9 to 13.8 V,
  about 0.03 V error), system current and battery charging current (mA), panel lid tamper,
  aux tamper, bell/reset activity, panel clock, keypad display text, event log pointer.
- **Event log**: an event entity (new events only) and a "Last event" sensor whose state is
  the event type name (for example `AlarmActive`, `BellTamper`) with description, group,
  parameter, areas and panel time as attributes. The last 10 events are published
  retained on `log/history` at start-up, so a restart never replays them as events.
- **Last user** sensor from Connect user messages (user number and method).
- **Text**: banner (16 characters, written to the panel's programming memory) and
  display message (32 characters via SetLCDDisplay).
- **Button**: sync the panel clock to the app's clock.

## MQTT topics

All topics start with `mqtt_prefix` (`texecom` by default).

| Topic | Direction | Payload |
|-------|-----------|---------|
| `status` | out | `online` / `offline` (retained, last will) |
| `area/<n>/state`, `area/<n>/attributes` | out | HA alarm state, JSON |
| `area/<n>/set` | in | `ARM_AWAY`, `ARM_HOME`, `ARM_NIGHT`, `ARM_VACATION`, `ARM_CUSTOM_BYPASS`, `DISARM` |
| `zone/<n>/state` | out | JSON: state, active, tamper, fault, alarmed, bypassed, text, areas |
| `zone/<n>/omit/set` | in | `ON` / `OFF` |
| `output/panel/<n>/state`, `output/digi/<n>/state` | out | `ON` / `OFF` |
| `pc_control/<n>/state`, `pc_control/<n>/set` | out / in | `ON` / `OFF` |
| `control_timer/<n>/state` | out | `ON` / `OFF` |
| `system/state` | out | JSON |
| `log/event` | out | JSON per new event (not retained) |
| `log/last`, `log/history` | out | Latest event / last 10 events at start-up (retained) |
| `user/last` | out | JSON |
| `banner/state`, `banner/set` | out / in | text |
| `lcd/state`, `lcd/set` | out / in | text |
| `cmd/reset/<area>` | in | any |
| `cmd/sync_clock` | in | any |
| `cmd/keypress` | in | keys separated by spaces: `0`-`9`, `omit`, `menu`, `yes`, `part`, `no`, `area`, `chime`, `reset`, `up`, `down` |

## Safety

- The UDL code is stored with the `password` schema type (masked in the UI). Home
  Assistant checks such values against the Have I Been Pwned list, and short numeric
  codes are always on it, so a "pwned secret" repair may appear. It can be dismissed.

- The app never reads user codes or prox tags and never logs the UDL code.
- UDL writes are refused unless the address is on the whitelist in
  `memory_map.py`, whatever arrives on MQTT.
- `read_only: true` is recommended for the first run against a new panel.
- Area numbers in Connect commands are 1-based (area A = 1), matching what the
  established bridge logs for the same panel.

## Support

Issues and source: https://github.com/Basskongen/texecom-hassio
