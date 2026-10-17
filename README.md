
# Texecom Alarm Bridge for Home Assistant

Connects a Texecom Premier Elite alarm panel to Home Assistant through the panel's
ComIP or SmartCom module. The app talks to the panel; its own integration brings the
panel into Home Assistant with alarm panels, sensors and a keypad card. No MQTT
broker needed.

[![Add repository to Home Assistant](https://my.home-assistant.io/badges/supervisor_add_addon_repository.svg)](https://my.home-assistant.io/redirect/supervisor_add_addon_repository/?repository_url=https%3A%2F%2Fgithub.com%2FBasskongen%2Ftexecom-alarm-bridge)

> **Experimental.** Tested on a Premier Elite 88 (firmware V5.06) with a ComIP.

## Features

- **Alarm panel per area**: arm (full and part arm 1–3) and disarm, with an optional code in
  Home Assistant.
- **Area status**: ready, arming, entry delay, armed, arm mode (Full, Part 1–3), fire alarm
  and intruder alarm.
- **Zones**: open/active, tamper and alarm memory for every zone in use, found automatically.
- **Panel**: panel and battery voltage, system current, battery charging current, battery and
  mains fault, lid and aux tamper, outputs (digi and panel), control timers, banner text and clock sync.
- **Event log**: every new panel event as a Home Assistant event.
- **Keypad card**: the panel's own display and keys, live in a dashboard.
- **Secure by design**: the app and the integration talk over a private, token-protected
  connection inside Home Assistant. Zones cannot be omitted and alarms cannot be reset
  from Home Assistant.

## Requirements

- Home Assistant OS or Supervised, 2026.3 or newer.
- A Texecom Premier Elite panel with a ComIP or SmartCom on a COM port, reachable on the network.
- The panel's **UDL passcode**, changed from the factory default (see below).
- No other program connected to the same ComIP/SmartCom: it accepts one client at a time.

## Installation

1. Select the button above, or add the repository by hand: **Settings > Apps > App store >
   ⋮ > Repositories**, and add `https://github.com/Basskongen/texecom-alarm-bridge`.
2. Find **Texecom Alarm Bridge** in the app store and select **Install**.
3. Open the **Configuration** tab and fill in at least:
   - `host`: the IP address of the ComIP / SmartCom
   - `udl_code`: the panel's UDL passcode
4. Start the app. The first start installs the Texecom integration and asks for a
   restart of Home Assistant.
5. After the restart, **Settings > Devices & services** shows **Texecom** as discovered.
   Select **Add**. The panel appears as a device named after its model.

### Change the UDL passcode first

It refuses to start with the factory default for safety reasons.

### Keep the UDL passcode in secrets.yaml (optional)

Add it to `secrets.yaml`:

```yaml
texecom_udl_code: "123456"
```

and write `!secret texecom_udl_code` in `udl_code` instead of the passcode itself.

## Configuration

### App options (Configuration tab)

| Option | | Example | What it does |
|---|---|---|---|
| `host` | required | `192.168.1.50` | IP address of the ComIP / SmartCom |
| `udl_code` | required | `123456` | The panel's UDL passcode |
| `port` | optional | `10001` | TCP port (default: 10001) |
| `max_zones` | optional | `48` | Highest zone number to look for (default: from the panel model) |
| `max_areas` | optional | `2` | Number of areas to read (default: from the panel model) |
| `part_arm_1` … `part_arm_3` | optional | `armed_night` | Home Assistant arm mode for each part arm (default: armed_home, armed_night, armed_vacation) |
| `zone_config` | optional | see the Documentation tab | Own name and device class per zone |
| `area_config` | optional | see the Documentation tab | Own name and part-arm modes per area |
| `keypad_enabled` | optional | `false` | Install the keypad card and let it operate the panel (default: on) |
| `show_panel_outputs` | optional | `true` | Show the panel's outputs (default: off) |
| `show_digi_outputs` | optional | `false` | Show the digi (communicator) outputs (default: on) |
| `show_control_timers` | optional | `true` | Show the control timers (default: off) |
| `read_only` | optional | `true` | Refuse everything that changes the panel (default: off) |

All options are described in the app's **Documentation** tab.

### Integration options

**Settings > Devices & services > Texecom > Configure**:

- **Ask for a code when disarming** and **when arming** (each optional).
- **Users**: each person their own code (user 1 is created at setup). Codes are stored
  only as a one-way hash. The alarm panel shows who armed or disarmed it.
- **Let all users use the keypad card** (off: administrators only).

### Keypad card

With `keypad_enabled` on (the default), the app installs the card and registers it as a dashboard resource.
Add it to a dashboard:

```yaml
type: custom:texecom-keypad-card
title: Alarm
haptic: true     # vibrate on key presses in the Home Assistant app
all_keys: false  # also show the Omit, Chime, Part and Area keys
style: nexus     # nexus or classic
```

## Support

Open an issue in this repository and include the app log.

---

This is an independent project. It is not made, endorsed or supported by Texecom.
Texecom and Premier Elite are trademarks of their owner and are used here only to
say which alarm panels the app works with.
