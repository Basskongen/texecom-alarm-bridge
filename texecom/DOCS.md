# Texecom Alarm Bridge

This is an independent project. It is not made, endorsed or supported by Texecom.
Texecom and Premier Elite are trademarks of their owner and are used here only to
say which alarm panels the app works with.

Bridges a Texecom alarm panel of the Premier Elite series to Home Assistant over MQTT with automatic
discovery. Tested against a Premier Elite 88 V5.06.00 with a ComIP on Com Port 1.

## How it works

The app opens one TCP connection to the ComIP/SmartCom and logs in twice on it:

- **Connect protocol**: live messages for zones, areas,
  outputs and users; arm, disarm and reset.
- **UDL protocol**: memory reads for voltages, tamper flags, the keypad
  display, the panel clock and the event log. Writes are limited to a short
  whitelist in the code (clock and banner text).

Only one client can be connected to the panel. Stop any other program that uses the connection
before starting the app. After a dropped connection the panel refuses new
connections for about a minute; the app waits and retries automatically.

## Before you start: change the UDL passcode

The app logs in with the panel's UDL passcode and can do everything the keypad can.
**It refuses to start with the factory default UDL passcode (1234).** Change it on the
panel first (engineer menu: UDL/Digi Options > UDL Options > UDL Password) to a code
nobody can guess, and use that code in the app settings.

## Installation

1. Add this repository in Settings > Apps > App store > three dots > Repositories.
2. Install "Texecom Alarm Bridge".
3. Fill in `host` and `udl_code`. MQTT details are taken from the Mosquitto app
   automatically; set `mqtt_host` only if you run another broker.
4. Start the app and watch the log. The device "Texecom alarm panel" appears
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
| `zones` | | Optional list of zones to read, e.g. `1-5, 74`. Skips searching through all zones and makes start-up faster |
| `max_areas` | 8 | Number of areas to read at start-up |
| `part_arm_1`, `part_arm_2`, `part_arm_3` | armed_home, armed_night, armed_vacation | Which Home Assistant arm state each part arm uses |
| `zone_config` | [] | Per-zone overrides: `number`, `name`, `device_class` |
| `area_config` | [] | Per-area overrides: `number`, `name`, `part_arm_1..3` |
| `legacy_prefix` | | MQTT prefix of the bridge this app replaces (see Migrating) |
| `keypad_enabled` | false | Pass key presses from the keypad card to the panel |
| `keypad_poll_interval` | 0.25 | Seconds between display reads while the keypad is enabled |
| `install_card` | true | Write the keypad card to `/config/www/texecom-keypad.js` at start-up |
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

## Virtual keypad

The card works like the panel's own keypad. The 2x16 display and the LEDs are read
4 times a second and sent straight to the card over MQTT, so the clock on line 2 does
not fill Home Assistant's history. The live view needs an admin user; other users see
line 1 and the LEDs from the entities.

Every key on the card (0-9, Yes, No, Part, Area, arrows, Omit, Menu, Chime, Reset) is
sent to the panel as it is pressed, exactly like the keypad on the wall. The panel's own
user codes and lockout protect it, as they do on the wall. Nothing is sent while
`read_only` is on or `keypad_enabled` is off.

Setup: enable `keypad_enabled` and restart the app. It writes `texecom-keypad.js` to
`/config/www` and registers it as a dashboard resource itself. Add the card to a dashboard:

```yaml
type: custom:texecom-keypad-card
prefix: texecom
title: Alarm
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
  tamper, alarm memory. Zones cannot be omitted from Home Assistant.
- **Outputs**: panel output 1-5 and digi output 1-8 (from Connect output messages).
- **Control Timer 1-8** sensors (read only).
- **System**: panel and battery voltage (calibrated against the panel's own diagnostics over 11.9 to 13.8 V,
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
| `output/panel/<n>/state`, `output/digi/<n>/state` | out | `ON` / `OFF` |
| `control_timer/<n>/state` | out | `ON` / `OFF` |
| `system/state` | out | JSON |
| `log/event` | out | JSON per new event (not retained) |
| `log/last`, `log/history` | out | Latest event / last 10 events at start-up (retained) |
| `user/last` | out | JSON |
| `banner/state`, `banner/set` | out / in | text |
| `lcd/state`, `lcd/set` | out / in | text |
| `cmd/reset/<area>` | in | any |
| `cmd/sync_clock` | in | any |
| `keypad/key` | in | one key: `0`-`9`, `yes`, `no`, `part`, `area`, `up`, `down`, `omit`, `menu`, `chime`, `reset` |
| `keypad/display` | out | JSON: both display lines and LEDs, on every change (not retained) |
| `keypad/state`, `keypad/result` | out | JSON: line 1 and LEDs (retained); errors from key presses |
| `cmd/read` | in | `<addr hex> <count> [prog]`: diagnostic memory read, only with `log_level: debug` |
| `cmd/keypress` | in | keys separated by spaces: `0`-`9`, `omit`, `menu`, `yes`, `part`, `no`, `area`, `chime`, `reset`, `up`, `down` |

## Safety

- Zones cannot be omitted from Home Assistant. Omitting is only possible from a keypad,
  the wall keypad or the keypad card, with a user code.
- Key presses and the UDL passcode are never written to the log, not even at debug level.
  At `info` the log only shows events in plain words.
- Key presses from the keypad card travel over MQTT on `<prefix>/keypad/key`. Anyone who
  can read that topic on the broker can see them. Restrict the broker so only Home
  Assistant and this app can use the `texecom/#` topics (Mosquitto app: `customize` with
  an ACL file), and keep `keypad_enabled` off if you do not need the card.

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
