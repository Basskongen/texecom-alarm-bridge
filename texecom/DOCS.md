# Texecom Alarm Bridge

This is an independent project. It is not made, endorsed or supported by Texecom.
Texecom and Premier Elite are trademarks of their owner and are used here only to
say which alarm panels the app works with.

Bridges a Texecom alarm panel of the Premier Elite series to Home Assistant. Tested
against a Premier Elite 88 V5.06.00 with a ComIP on Com Port 1.

## How it works

The app opens one TCP connection to the ComIP/SmartCom and logs in twice on it:

- **Connect protocol**: live messages for zones, areas,
  outputs and users; arm and disarm.
- **UDL protocol**: memory reads for voltages, tamper flags, the keypad
  display, the panel clock and the event log. Writes are limited to a short
  whitelist in the code (clock and banner text).

Home Assistant gets everything through the **Texecom integration**, which the app installs
itself. The app and the integration talk over a private connection on the Supervisor's
internal network, protected by a token the app creates; nothing goes through MQTT unless
you switch MQTT on.

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
3. Fill in `host` and `udl_code`, then start the app.
4. The first start installs the Texecom integration and shows a notification: restart
   Home Assistant (Settings > System > Restart).
5. After the restart, Settings > Devices & services shows **Texecom** as discovered.
   Select **Add**. The device, named after the panel model (for example
   "Texecom Premier Elite 88"), appears under the Texecom integration.

### Integration options

Settings > Devices & services > Texecom > Configure:

| Option | Default | Description |
|--------|---------|-------------|
| Code for disarming in Home Assistant | empty | Asked for when an area is disarmed from Home Assistant. A Home Assistant code, not a panel user code. Empty = no code |
| Also ask for the code when arming | off | Only used when a code is set |
| Let all users use the keypad card | off | Off: only administrators can use the keypad card |

## Options

The first start searches zone 1 to `max_zones` and remembers the zones in use. Later
starts read only those, so start-up is quick, and then look through the other zone
numbers in the background: a zone taken into use in the panel appears in Home Assistant
a little after start-up, and a zone taken out of use disappears.


| Option | Default | Description |
|--------|---------|-------------|
| `host` | | IP address of the ComIP / SmartCom |
| `port` | 10001 | TCP port |
| `udl_code` | | UDL passcode (4 to 6 digits). Used for both logins |
| `mqtt_enabled` | false | Also publish to MQTT (see MQTT below). Off: nothing goes through the broker |
| `mqtt_host`, `mqtt_port`, `mqtt_username`, `mqtt_password` | from Supervisor | Override the broker (MQTT only) |
| `mqtt_prefix` | `texecom` | Topic prefix for state and command topics (MQTT only) |
| `discovery_prefix` | `homeassistant` | MQTT discovery prefix (MQTT only) |
| `max_zones` | 88 | Highest zone number to look for |
| `max_areas` | 8 | Number of areas to read at start-up |
| `part_arm_1`, `part_arm_2`, `part_arm_3` | armed_home, armed_night, armed_vacation | Which Home Assistant arm state each part arm uses |
| `zone_config` | [] | Per-zone overrides: `number`, `name`, `device_class` |
| `area_config` | [] | Per-area overrides: `number`, `name`, `part_arm_1..3` |
| `legacy_prefix` | | Prefix of the bridge this app replaces (see Migrating) |
| `keypad_enabled` | false | Pass key presses from the keypad card to the panel |
| `keypad_poll_interval` | 0.25 | Seconds between display reads while the keypad is enabled |
| `install_card` | true | Write the keypad card to `/config/www/texecom-keypad.js` at start-up |
| `show_panel_outputs` | false | Show panel output 1-5 as entities |
| `show_control_timers` | false | Show control timer 1-8 as entities |
| `poll_interval` | 5 | Seconds between memory polls |
| `read_only` | false | Refuse every command that changes the panel |
| `allow_keypress` | false | Accept virtual key presses on `texecom/cmd/keypress` (MQTT only) |
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
4 times a second and sent straight to the card through the Texecom integration, so the
clock on line 2 does not fill Home Assistant's history. Key presses go from the card to
Home Assistant (logged-in user) and on to the app over the private connection, never
through MQTT. Only administrators can use the card, unless the integration option
allows it for all users.

Every key on the card (0-9, Yes, No, Part, Area, arrows, Omit, Menu, Chime, Reset) is
sent to the panel as it is pressed, exactly like the keypad on the wall. The panel's own
user codes and lockout protect it, as they do on the wall. Each press flashes the key and vibrates the phone
in the Home Assistant app. The keys are laid out like the panel's keypad. Nothing is sent while
`read_only` is on or `keypad_enabled` is off.

Setup: enable `keypad_enabled` and restart the app. It writes `texecom-keypad.js` to
`/config/www` and registers it as a dashboard resource itself. Add the card to a dashboard:

```yaml
type: custom:texecom-keypad-card
title: Alarm
haptic: true   # vibrate on key presses in the Home Assistant app
all_keys: false   # also show the Omit, Chime, Part and Area keys
```

The card can also be set up in the dashboard's visual editor.

## Migrating from earlier versions and other bridges

From 0.3.0 Home Assistant gets the panel through the Texecom integration instead of MQTT.
With MQTT off, the app removes the MQTT entities of earlier versions from the broker at
its first start, and the integration creates new ones. Automations, scripts and
dashboards that used the old entities must be pointed at the new ones.

If Home Assistant had entities from a previous Texecom-to-MQTT bridge, `legacy_prefix`
(the prefix that bridge's topics started with) keeps their unique ids and the device
(identified by the panel serial number) inside the integration. With `mqtt_enabled` on,
the app also:

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
  disarm.
- **Binary sensors** per used zone: open/active (device class from the zone type),
  tamper, alarm memory. Zones cannot be omitted from Home Assistant.
- **Outputs**: digi output 1-8, and panel output 1-5 with `show_panel_outputs`, as the
  outputs really are (inverted outputs included). Read from the panel's memory at start-up
  and every poll, and updated at once from the panel's output messages.
- **Control Timer 1-8** sensors (read only), with `show_control_timers`.
- **Keypad indicators**: armed, alarm, fire and tamper, as the panel signals them to its
  keypads.
- **System**: panel and battery voltage (calibrated against the panel's own diagnostics over 11.9 to 13.8 V,
  about 0.03 V error), system current and battery charging current (mA), panel lid tamper,
  aux tamper, panel heartbeat, keypad display text.
- **Event log**: an event entity (new events only) whose event type is the event name
  (for example `AlarmActive`, `BellTamper`) with description, group, parameter, areas and
  panel time as attributes. The latest event is also published retained on `log/last`. The last 10 events are published
  retained on `log/history` at start-up, so a restart never replays them as events.
- **Last user** sensor from Connect user messages (user number and method).
- **Text**: banner (16 characters, written to the panel's programming memory). A
  temporary display message (32 characters) can be sent on the `lcd/set` topic.
- **Button**: sync the panel clock to the app's clock.

## MQTT (optional)

Off by default. With `mqtt_enabled: true` the app also publishes everything to the MQTT
broker with Home Assistant MQTT discovery, as versions before 0.3.0 did. Anyone with access
to the broker can then see the panel and send commands, including disarm: restrict the
broker (Mosquitto app: `customize` with an ACL file). All topics start with `mqtt_prefix`
(`texecom` by default).

| Topic | Direction | Payload |
|-------|-----------|---------|
| `status` | out | `online` / `offline` (retained, last will) |
| `area/<n>/state`, `area/<n>/attributes` | out | HA alarm state, JSON |
| `area/<n>/set` | in | `ARM_AWAY`, `ARM_HOME`, `ARM_NIGHT`, `ARM_VACATION`, `ARM_CUSTOM_BYPASS`, `DISARM` |
| `zone/<n>/state` | out | JSON: state, active, tamper, fault, alarmed, bypassed, text, areas |
| `output/panel/<n>/state`, `output/digi/<n>/state` | out | `ON` / `OFF` |
| `control_timer/<n>/state` | out | `ON` / `OFF` |
| `system/state` | out | JSON |
| `heartbeat/state` | out | JSON: `running`, `period` (seconds per blink) |
| `log/event` | out | JSON per new event (not retained) |
| `log/last`, `log/history` | out | Latest event / last 10 events at start-up (retained) |
| `user/last` | out | JSON |
| `banner/state`, `banner/set` | out / in | text |
| `lcd/state`, `lcd/set` | out / in | text |
| `cmd/sync_clock` | in | any |
| `keypad/key` | in | one key: `0`-`9`, `yes`, `no`, `part`, `area`, `up`, `down`, `omit`, `menu`, `chime`, `reset` |
| `keypad/display` | out | JSON: both display lines and LEDs, on every change (not retained) |
| `keypad/state` | out | JSON: line 1 and LEDs (retained) |
| `keypad/result` | out | JSON: errors from key presses, shown on the card (not retained) |
| `cmd/read` | in | `<addr hex> <count> [prog]`: diagnostic memory read, only with `log_level: debug` |
| `cmd/keypress` | in | keys separated by spaces: `0`-`9`, `omit`, `menu`, `yes`, `part`, `no`, `area`, `chime`, `reset`, `up`, `down` |

## Safety

- Zones cannot be omitted and alarms cannot be reset from Home Assistant. Both are only
  possible from a keypad, the wall keypad or the keypad card, with a user code.
- Key presses and the UDL passcode are never written to the log, not even at debug level.
  At `info` the log only shows events in plain words.
- The app and the integration talk over a private connection on the Supervisor's internal
  network with a token the app creates and keeps in its own data folder. The app maps no
  ports to the host. Key presses and commands never go through MQTT unless `mqtt_enabled`
  is on.
- The keypad card is for administrators only unless the integration option allows all
  users. Disarming from Home Assistant can require a code (integration option).

- The UDL code is stored with the `password` schema type (masked in the UI). Home
  Assistant checks such values against the Have I Been Pwned list, and short numeric
  codes are always on it, so a "pwned secret" repair may appear. It can be dismissed.

- The app never reads user codes or prox tags and never logs the UDL code.
- UDL writes are refused unless the address is on the whitelist in
  `memory_map.py`, whatever command arrives.
- `read_only: true` is recommended for the first run against a new panel.
- Area numbers in Connect commands are 1-based (area A = 1), matching what the
  established bridge logs for the same panel.

## Support

Issues and source: https://github.com/Basskongen/texecom-hassio
