<img src="texecom/icon.png" alt="" width="96" align="right">

# Texecom Alarm Bridge for Home Assistant

Connects a Texecom Premier Elite alarm panel to Home Assistant through the panel's
ComIP or SmartCom module. The app talks to the panel; its own integration brings the
panel into Home Assistant with alarm panels, sensors and a virtual keypad. No MQTT
broker needed.

[![Add repository to Home Assistant](https://my.home-assistant.io/badges/supervisor_add_addon_repository.svg)](https://my.home-assistant.io/redirect/supervisor_add_addon_repository/?repository_url=https%3A%2F%2Fgithub.com%2FBasskongen%2Ftexecom-hassio)

> **Experimental.** Tested on a Premier Elite 88 (firmware V5.06) with a ComIP.
> Start with `read_only: true` on a new panel.

## Features

- **Alarm panel per area**: arm (full and part arm 1–3) and disarm, with an optional code in
  Home Assistant.
- **Area status**: ready, arming, entry delay, armed, arm mode (Full, Part 1–3), fire alarm
  and intruder alarm.
- **Zones**: open/active, tamper and alarm memory for every zone in use, found automatically.
- **Panel**: panel and battery voltage, system current, battery charging current, battery and
  mains fault, lid and aux tamper, outputs (digi and panel), control timers, banner text and clock sync.
- **Event log**: every new panel event as a Home Assistant event.
- **Virtual keypad card**: the panel's own display and keys, live in a dashboard.
- **Secure by design**: the app and the integration talk over a private, token-protected
  connection inside Home Assistant. Zones cannot be omitted and alarms cannot be reset
  from Home Assistant.

## Requirements

- Home Assistant OS or Supervised, 2026.3 or newer.
- A Texecom Premier Elite panel with a ComIP or SmartCom on a COM port, reachable on the network.
- The panel's **UDL passcode**, changed from the factory default (see below).
- No other program connected to the panel: it accepts one client at a time.

## Installation

1. Select the button above, or add the repository by hand: **Settings > Apps > App store >
   ⋮ > Repositories**, and add `https://github.com/Basskongen/texecom-hassio`.
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

## Configuration

### App options (Configuration tab)

| Option | Default | What it does |
|---|---|---|
| `host`, `port` | –, 10001 | Address of the ComIP / SmartCom |
| `udl_code` | – | UDL passcode (4 to 6 digits) |
| `max_zones` | 88 | Highest zone number to look for. Found zones are remembered |
| `max_areas` | 8 | Number of areas to read |
| `part_arm_1` … `part_arm_3` | home, night, vacation | Which Home Assistant arm mode each part arm uses |
| `zone_config`, `area_config` | – | Own names and device classes for zones, names and part-arm modes for areas |
| `keypad_enabled` | off | Let the keypad card send key presses to the panel |
| `show_panel_outputs`, `show_control_timers` | off | Show panel outputs 1–5 and control timers 1–8 |
| `read_only` | off | Refuse everything that changes the panel |

All options are described in the app's **Documentation** tab.

### Integration options

**Settings > Devices & services > Texecom > Configure**:

- **Ask for a code when disarming** and **when arming** (each optional).
- **Users**: each person their own code (user 1 is created at setup). Codes are stored
  only as a one-way hash. The alarm panel shows who armed or disarmed it.
- **Let all users use the keypad card** (off: administrators only).

### Keypad card

With `keypad_enabled` on, the app installs the card and registers it as a dashboard resource.
Add it to a dashboard:

```yaml
type: custom:texecom-keypad-card
title: Alarm
haptic: true     # vibrate on key presses in the Home Assistant app
all_keys: false  # also show the Omit, Chime, Part and Area keys
```

## Support

Open an issue in this repository and include the app log.

---

This is an independent project. It is not made, endorsed or supported by Texecom.
Texecom and Premier Elite are trademarks of their owner and are used here only to
say which alarm panels the app works with.
