# Texecom Alarm Bridge

Connects a Texecom Premier Elite alarm panel (via ComIP or SmartCom) to Home Assistant.
The app talks to the panel and installs its own Texecom integration, which brings the
panel into Home Assistant. No MQTT broker needed.

**What you get**

- An alarm panel per area: full and part arm, disarm (optional code in Home Assistant)
- Area status: ready, arming, entry delay, armed, arm mode, fire alarm, intruder alarm
- Every zone in use: open/active, tamper and alarm memory
- Voltages, currents, battery and mains fault, tamper, outputs, event log, banner text
  and clock sync
- A keypad card with the panel's own display and keys

**Getting started**

1. Change the panel's UDL passcode from the factory default. The app refuses to start
   with it for safety reasons.
2. Fill in `host` and `udl_code` on the **Configuration** tab and start the app.
3. Restart Home Assistant when the notification asks for it.
4. Add **Texecom** under Settings > Devices & services (it is discovered automatically).

A ComIP/SmartCom accepts one client at a time: stop anything else that uses the one
the app connects to. Full instructions are on the **Documentation** tab.

This is an independent project. It is not made, endorsed or supported by Texecom.
Texecom and Premier Elite are trademarks of their owner and are used here only to
say which alarm panels the app works with.
