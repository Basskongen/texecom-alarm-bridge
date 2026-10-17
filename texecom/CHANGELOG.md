# Changelog

## 0.3.12

- Version bump of the app and the integration, to try the new integration update.

## 0.3.11

- A new version of the integration waits under Settings > Updates. Install puts it in
  place without restarting; Repairs then shows "Restart required".

## 0.3.10

- New option `use_connect` (on by default). Connect reports changes instantly; when it is
  off, the app reacts more slowly.

## 0.3.9

- A wrong UDL passcode is never retried: the app logs in once and then waits until the
  passcode is corrected and the app restarted.
- The UDL passcode is a plain text field instead of a password field, so Home Assistant no
  longer reports it as a leaked password. `!secret name` takes it from `secrets.yaml`.

## 0.3.8

- Turning the keypad off removes the keypad card and its dashboard resource again.

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
