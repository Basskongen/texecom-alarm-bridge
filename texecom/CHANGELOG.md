# Changelog

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
