# Changelog

## 0.3.0

- **Home Assistant gets the panel through the new Texecom integration instead of MQTT.**
  The app installs the integration itself and asks for a Home Assistant restart; after
  that Home Assistant offers "Texecom" under Devices & services. The app and the
  integration talk over a private, token-protected connection on the Supervisor's
  internal network. Commands, key presses and the panel's state no longer go through
  the MQTT broker.
- New entities (new entity ids) replace the MQTT ones, which the app removes from the
  broker at its first start. Automations and dashboards must be pointed at the new
  entities.
- Integration options: a code for disarming from Home Assistant (optional, also for
  arming if wanted), and whether all users or only administrators may use the keypad card.
- The keypad card talks to the integration instead of MQTT; the `prefix` setting is gone.
- MQTT is now optional (`mqtt_enabled`, off by default); the Mosquitto app is no longer
  needed.

## 0.2.5

- The app remembers the zones it finds. Later starts read only those and look for new
  zones in the background, so start-up is quick and new zones still appear by themselves.
- The `zones` setting from 0.2.4 is removed; it is no longer needed.
- Keypad card: all keys the same size, Menu grey so it is easy to find. Omit, Chime,
  Part and Area are shown only with the new `all_keys` setting ("Show all keys").
- Keypad card: the display is centred in the card (the 16-character lines keep the panel's
  own layout).

## 0.2.4

- Texts and documentation no longer name other programs.
- New optional setting `zones` (e.g. `1-5, 74`): read only these zones instead of
  searching through all of them at start-up.
- Keypad card: feedback on key presses (the key flashes and the phone vibrates in the
  Home Assistant app).
- Keypad card: keys laid out like the panel's keypad, all texts in English.
- The device is named after the panel model it reports, for example "Texecom Premier
  Elite 88" (was "Texecom alarm panel"). "Texecom alarm" if the panel does not say.
- The area reset button is removed for security: it reset alarms without a user code.
  Reset from a keypad or the keypad card with a code.
- "Bell / reset activity" is replaced by "Panel heartbeat". The bit it read blinks (most
  likely the heartbeat LED on the panel board); it is now read 4 times a second and the
  sensor shows whether it blinks, with the blink time as an attribute.
- Panel output 1-5 and Control timer 1-8 are hidden unless `show_panel_outputs` /
  `show_control_timers` is turned on.
- Removed the "Panel clock" and "Event log pointer" sensors.
- Keypad LED entities: "power" and "ready" are removed (the keypad card still shows
  them). "armed", "alarm", "fire" and "tamper" are now "Keypad ... indicator" with new
  entity ids (`binary_sensor.texecom_keypad_armed_indicator` and so on).
- The "Display message" entity is removed; the `lcd/set` topic still works.
- The "Last event" sensor is switched off for now; the "Panel event" entity remains.
- Keypad card: `haptic` setting (on by default) and support for the visual card editor.
- Keypad card: errors are shown straight from MQTT and clear after 5 seconds. The
  "Keypad result" sensor is removed.

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

- Keypad display is read 4 times a second while the keypad is enabled.
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
