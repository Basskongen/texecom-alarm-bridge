# Texecom Alarm Bridge

Connects a Texecom alarm panel of the Premier Elite series (via ComIP or SmartCom) to Home Assistant
over MQTT with automatic discovery.

Areas appear as alarm panels, zones as binary sensors, plus outputs,
panel and battery voltage, tamper flags, LCD text and the event log.

Uses both the Connect protocol (live events, arm/disarm) and the Wintex/UDL
protocol (memory reads) on a single TCP connection. Only one client can talk to
the panel at a time: stop texecom2mqtt and Wintex before starting this app.

**Experimental.** Test it with `read_only: true` first.

**Change the panel's UDL passcode before installing.** The app refuses to start with
the factory default (1234), because anyone who knows the UDL passcode can control the panel.

This is an independent project. It is not made, endorsed or supported by Texecom.
Texecom and Premier Elite are trademarks of their owner and are used here only to
say which alarm panels the app works with.
