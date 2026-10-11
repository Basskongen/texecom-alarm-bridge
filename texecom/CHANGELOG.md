# Changelog

## 0.2.3

- The app registers the keypad card as a dashboard resource itself, with its version in
  the URL so phones and browsers load the new card after an update. Needs access to the
  Home Assistant API (`homeassistant_api`). Old manual registrations are replaced.

## 0.2.2

- Voltages are shown with two decimals (13.72 V instead of 14 V).
- The keypad card sends every key as it is pressed, like the wall keypad, with all keys
  (including Omit, Menu, Chime and Reset). The code + one key request is gone.
- The app refuses to start with the factory default UDL passcode.
- Zone omit switches removed: zones can no longer be omitted from Home Assistant.
- PC Control switches removed: Home Assistant can no longer drive panel outputs.
- Zone tamper sensors are only created for zones wired with end-of-line resistors;
  normally closed zones (no tamper state) no longer get one.
- Key presses are never logged, not even at debug level. Diagnostic memory reads only
  work with `log_level: debug`.
- Sensors that earlier versions created and this one no longer does (the old LCD
  sensor) are removed from Home Assistant automatically.
- Renamed to Texecom Alarm Bridge and marked experimental. Independent project, not affiliated
  with Texecom; the device in Home Assistant is now called "Texecom alarm panel"
  unless the legacy mode keeps the old device.

## 0.2.1

- Keypad display is read 4 times a second (like Wintex) while the keypad is enabled.
- The card gets the live display straight from MQTT, so the per-second clock on line 2
  is not written to Home Assistant's history. Entities keep line 1 and the LEDs.
- The "Keypad display" entity replaces the system LCD sensor and shows line 1 only.

## 0.2.0

- Virtual keypad: a Lovelace card (`texecom-keypad.js`, written to `/config/www` by the
  app) shows the panel's display and LEDs live and sends a 4-digit code plus one action
  key. The app sends No, No, the code and the key, checks "Kode Akseptert" and the user
  name on the display, refuses Master/Install codes, blocks menu, reset and omit, and
  allows 3 attempts per minute. Off by default (`keypad_enabled`).
- Keypad display and LED entities, refreshed every second while the keypad is enabled.

## 0.1.3

- Login no longer fails when the panel sends a Connect log message inside the UDL
  login reply (happens when event messages were left on by a previous session).
- Event messages are switched off before hanging up, and the receive buffer is
  resynchronised after a timeout.

## 0.1.2

- Diagnostic read command on `<prefix>/cmd/read` (payload `<addr hex> <count> [prog]`),
  result on `<prefix>/debug/read` and in the log. Read-only; user code ranges are refused.

## 0.1.1

- Output sensors stay unknown until the panel reports the output group, instead of
  showing a wrong "off" for outputs that are on at start-up (for example inverted outputs).
- Zone names from two-line panel texts no longer carry the padding between the lines.
- First version verified against a Premier Elite 88 in Home Assistant.

## 0.1.0

- Per-zone and per-area overrides, part-arm mapping, and an optional legacy mode
  that reuses the entity ids, device and topics of the bridge being replaced.
- Power values (volts, mA) from the panel's power report; serial number from the UDL probe.

- First version: Connect + UDL on one connection, MQTT discovery for areas,
  zones, outputs, PC Control, system status and event log.
