# Texecom Alarm Bridge

This is an independent project. It is not made, endorsed or supported by Texecom.
Texecom and Premier Elite are trademarks of their owner and are used here only to
say which alarm panels the app works with.

Connects a Texecom alarm panel of the Premier Elite series to Home Assistant through the
panel's ComIP or SmartCom. Tested on a Premier Elite 88 (firmware V5.06) with a ComIP.

## How it works

The app talks to the panel and installs its own **Texecom integration** in Home
Assistant. The app and the integration talk over a private, token-protected connection
inside Home Assistant; nothing is exposed on your network.

Only one program can be connected to the panel at a time. Stop anything else that uses
the ComIP/SmartCom before starting the app. After a dropped connection the panel refuses
new connections for about a minute; the app waits and retries by itself.

## Before you start: change the UDL passcode

The app refuses to start with the factory default UDL passcode for safety reasons.
Change it on the panel first and use the new code in the app settings.

## Installation

1. Add this repository in Settings > Apps > App store > ⋮ > Repositories.
2. Install **Texecom Alarm Bridge**.
3. Fill in `host` and `udl_code`, then start the app.
4. The first start installs the Texecom integration and asks for a restart of Home
   Assistant.
5. After the restart, Settings > Devices & services shows **Texecom** as discovered.
   Select **Add**. The panel appears as a device named after its model.

## App options

Zones in use are found on the first start and remembered. Later starts look for added or
removed zones in the background.

| Option | Default | Description |
|--------|---------|-------------|
| `host` | | IP address of the ComIP / SmartCom |
| `port` | 10001 | TCP port |
| `udl_code` | | UDL passcode (4 to 6 digits) |
| `max_zones` | 88 | Highest zone number to look for |
| `max_areas` | 8 | Number of areas to read |
| `part_arm_1`, `part_arm_2`, `part_arm_3` | armed_home, armed_night, armed_vacation | Which Home Assistant arm mode each part arm uses |
| `zone_config` | [] | Per zone: `number`, `name`, `device_class` |
| `area_config` | [] | Per area: `number`, `name`, `part_arm_1..3` |
| `keypad_enabled` | false | Let the keypad card operate the panel; installs the card |
| `keypad_poll_interval` | 0.25 | Seconds between display reads while the keypad is enabled |
| `show_panel_outputs` | false | Show panel output 1-5 |
| `show_control_timers` | false | Show control timer 1-8 |
| `poll_interval` | 5 | Seconds between status reads |
| `read_only` | false | Refuse everything that changes the panel |
| `log_level` | info | Amount of logging |

Example:

```yaml
zone_config:
  - number: 5
    name: Front door
    device_class: door
area_config:
  - number: 1
    name: House
    part_arm_1: armed_night
    part_arm_2: armed_home
```

## Integration options

Settings > Devices & services > Texecom > Configure has **Settings**, **Add user**,
**Change user** and **Remove user**. Settings:

| Option | Default | Description |
|--------|---------|-------------|
| Ask for a code when disarming | off | A user's code is needed to disarm from Home Assistant |
| Ask for a code when arming | off | A user's code is needed to arm from Home Assistant |
| Let all users use the keypad card | off | Off: only administrators can use the keypad card |

### Users and codes

Each person can have their own Home Assistant code (not a panel user code). User 1 is
created at setup. Codes are stored only as a one-way hash. The alarm panel shows who armed or disarmed it.

## Updates

Updating the app also updates the integration. Settings > Updates then shows
**Texecom integration**; **Install** restarts Home Assistant to load it.

## Entities

Entity ids are fixed and never taken from names: the alarm panel for area A is
`alarm_control_panel.texecom_alarm` (area B `texecom_alarm_b` …), zones are
`binary_sensor.texecom_zone_<n>`.

- **Alarm panel** per area with zones: full arm, part arm 1–3 and disarm. States:
  disarmed, arming, pending, armed (as set with `part_arm_*`), triggered.
- **Per area**: ready (can be armed), arming (exit delay), entry delay, armed, arm mode
  (Disarmed, Full, Part 1–3), fire alarm, intruder alarm.
- **Per zone in use**: open/active, tamper (not for normally closed loops such as fire
  zones) and alarm memory.
- **Outputs**: digi output 1–8, and panel output 1–5 with `show_panel_outputs`, as the
  outputs really are (inverted outputs included). Control timer 1–8 with
  `show_control_timers`.
- **Panel**: panel and battery voltage, system current, battery charging current, lid
  tamper, aux tamper, battery fault, mains fault (reported after the panel's mains fail
  delay; the event log shows "AC Fail" much sooner), keypad display and indicators.
- **Panel event**: an event entity for every new entry in the panel's event log.
- **Banner text** (16 characters) and a **Sync panel clock** button.

## Keypad card

The card shows the panel's display and lamps live and sends each key to the panel as it
is pressed. Only administrators can use it unless the integration option allows all
users. Nothing is sent while `read_only` is on or `keypad_enabled` is off.

Enable `keypad_enabled` and restart the app; it installs the card itself:

```yaml
type: custom:texecom-keypad-card
title: Alarm
haptic: true      # vibrate on key presses in the Home Assistant app
all_keys: false   # also show the Omit, Chime, Part and Area keys
style: nexus      # nexus or classic
```

## Safety

- Zones cannot be omitted and alarms cannot be reset from Home Assistant.
- Key presses and the UDL passcode are never written to the log.
- The app never reads user codes or prox tags.
- `read_only: true` is recommended for the first run against a new panel.
- A "pwned secret" repair for the UDL passcode can be dismissed (short numeric codes
  are always on the leaked-password list).

## Support

Open an issue at https://github.com/Basskongen/texecom-hassio. Include the app log.
