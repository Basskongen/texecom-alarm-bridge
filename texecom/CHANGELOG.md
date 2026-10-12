# Changelog

## 0.3.7

- New option `show_digi_outputs` (on by default) next to `show_panel_outputs`.
- The keypad is on by default (`keypad_enabled`).
- `max_zones` and `max_areas` are optional: left empty, they come from the panel model
  (for example 48 zones and 4 areas on a Premier Elite 48).
- After an update, Repairs shows "Restart required" like other integrations; fixing it
  restarts Home Assistant. The restart notification is no longer used for updates.
- The app store repository is now `texecom-alarm-bridge`.
- Option tables show required/optional and an example; each option says its default.

## 0.3.6

First public beta.

- Alarm panel per area: full arm, part arm 1–3 and disarm, with optional user codes in
  Home Assistant.
- Area status: ready, arming, entry delay, armed, arm mode, fire alarm, intruder alarm.
- Every zone in use: open/active, tamper and alarm memory.
- Panel: voltages, currents, battery and mains fault, tamper, outputs, event log, banner
  text and clock sync.
- Keypad card with the panel's own display and keys, in a Nexus or classic style.
- Its own Home Assistant integration, installed and updated by the app.
